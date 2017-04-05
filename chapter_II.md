Welcome back dear developers, good to see you here.

In the first chapter, you saw how to use Vue inside your Twig templates
and how to communicate with Symfony, in this chapter, we gonna dive deeper
into the Vue logic and make him really more "close" to Symfony.

## Part I - Did you say Ajax ?

Alright, now, time to get in the serious business of Vue inside of Symfony,
for this, we gonna update our form and make the HTTP request in order to
show the results.

In the earlier time of web, you may use Jquery or even pure JS for
Ajax call, let's be clear, this time is out !
In fact, Vue doesn't allow HTTP call in the "classic" version of the framework,
for this, you must implement a library called vue-resource, i prefer to be clear,
this library was part of the ecosystem of Vue but the creator of Vue say
that Vue must be concentrate on the core functionality and this library was
deprecated, in fact, the control of the library was passed to a new team
and Vue's gonna implement some functionality that come with vue-resource
later.

Here, we gonna implement this library and do simple HTTP call from our frontend
to our backend, this way, our application feel more "actual" and faster.

Alright, time to get serious, for implementing vue-resource, let's open
github and the repo of the library, once this's done, let's implement
the script tag in our main layout and build our first HTTP call !

```twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>{% block title %}Vue in Symfony !{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
        <link rel="icon" type="image/x-icon" href="{{ asset('favicon.ico') }}" />
    </head>
    <body>
        <div id="app">
            {% block body %}{% endblock %}
        </div>
        {% block javascripts %}
            <script src="https://unpkg.com/vue"></script>
            <script src="https://cdn.jsdelivr.net/vue.resource/1.2.1/vue-resource.min.js"></script>
        {% endblock %}
    </body>
</html>
```

For simplicity, i use PHPStorm so i've decide to download the library
(with Ctrl + Enter -> Download library).

Once this's done, let's update our products view and inject HTTP calls.
First, we gonna add a new components, this components gonna "contains" all
the logic of showing and adding some functionality like routing to the details page
via Vue.
This way, the component gonna contains some attributes and Symfony gonna
"inject" this attributes from Twig.

Alright, let's update our view linked to the products and add somme HTTP calls :

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% for products in products %}
        <p>{{ products.name }}</p>
    {% endfor %}
    {{ form_start(searchForm, {'attr': {'v-on:submit.prevent': 'searchProducts'}}) }}
    {{ form_label(searchForm.name) }}
        {{ form_widget(searchForm.name, {'attr': {'v-model': 'name'}}) }}
        {{ form_errors(searchForm.name) }}
    <button type="submit">Rechercher</button>
    {{ form_end(searchForm) }}
    {% verbatim %}
        <p v-if="name">
            How, seems like you're searching {{ name }},
            here's the list of products wih this category that we have in stock :
        </p>
    {% endverbatim %}
{% endblock %}

{% block javascripts %}
    {{ parent() }}
    <script>
        var app = new Vue({
            el: '#app',
            data: {
                name: '',
                apiURL: '',
                products: []
            },
            methods: {
                searchProducts: function() {
                    this.$http.get(this.apiURL, this.name).then(response => {
                        return response.json();
                    }).then(data => {
                        for (let key in data) {
                            this.products.push(data[key]);
                        }
                    });
                }
            }
        })
    </script>
{% endblock %}
```

Alright, what's new here ? First, we add some variables
in order to store the results and being able to call the API then
we define a method who's gonna been call once we submit the form,
this way, we parse the response and push every results into the products
array stored into our Vue instance.

Problem is, where's our API return ?

We gonna build it now, directly from scratch without using Api Platform
for example, that's not so complicated, first, let's update our Entity :

```php
<?php

/*
 * This file is part of the SymfonyVue project.
 *
 * (c) Guillaume Loulier <contact@guillaumeloulier.fr>
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 */

namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * Class Products
 *
 * @author Guillaume Loulier <contact@guillaumeloulier.fr>
 *
 * @ORM\Entity
 * @ORM\Table(name="products")
 */
class Products
{
    /**
     * @var int
     *
     * @ORM\Id
     * @ORM\Column(name="id")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @var string
     *
     * @ORM\Column(name="name", type="string", length=200, nullable=false)
     */
    private $name;

    /**
     * @var string
     *
     * @ORM\Column(name="price", type="string", length=50, nullable=false)
     */
    private $price;

    /**
     * @var string
     *
     * @ORM\Column(name="category", type="string", length=100, nullable=false)
     */
    private $category;

    /**
     * @return int
     */
    public function getId () : int
    {
        return $this->id;
    }

    /**
     * @return string
     */
    public function getName () : string
    {
        return $this->name;
    }

    /**
     * @param $name
     *
     * @return $this
     */
    public function setName ($name)
    {
        $this->name = $name;

        return $this;
    }

    /**
     * @return string
     */
    public function getPrice () : string
    {
        return $this->price;
    }

    /**
     * @param $price
     *
     * @return $this
     */
    public function setPrice ($price)
    {
        $this->price = $price;

        return $this;
    }

    /**
     * @return string
     */
    public function getCategory () : string
    {
        return $this->category;
    }

    /**
     * @param string $category
     *
     * @return $this
     */
    public function setCategory (string $category)
    {
        $this->category = $category;

        return $this;
    }
}
```

Here, we simply add a category attribute, we gonna launch the research
via the category, for simplicity.
Now, let's update our manager :

```php
<?php

/*
 * This file is part of the SymfonyVue project.
 *
 * (c) Guillaume Loulier <contact@guillaumeloulier.fr>
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 */

namespace AppBundle\Managers;

use AppBundle\Entity\Products;
use AppBundle\Form\Type\SearchProductsType;
use Doctrine\ORM\EntityManager;
use Symfony\Component\Form\FormFactory;
use Symfony\Component\HttpFoundation\RequestStack;
use Symfony\Component\OptionsResolver\Exception\InvalidOptionsException;

/**
 * Class ProductsManager
 *
 * @author Guillaume Loulier <contact@guillaumeloulier.fr>
 */
class ProductsManager
{
    /** @var EntityManager */
    private $doctrine;

    /** @var FormFactory */
    private $form;

    /** @var RequestStack */
    private $request;

    /**
     * ProductsManager constructor.
     *
     * @param EntityManager $doctrine
     * @param FormFactory   $form
     * @param RequestStack  $request
     */
    public function __construct (
        EntityManager $doctrine,
        FormFactory $form,
        RequestStack $request
    ) {
        $this->doctrine = $doctrine;
        $this->form = $form;
        $this->request = $request;
    }

    /**
     * @return array
     */
    public function getAllProducts()
    {
        return $this->doctrine->getRepository(Products::class)->findAll();
    }

    /**
     * @param string $name
     *
     * @return array
     */
    public function getProductsByName(string $name)
    {
        $products = $this->doctrine->getRepository(Products::class)
                                   ->findBy(['category' => $name]);

        $data = [];
        foreach ($products as $product) {
            $data = [
                'name' => $product->getName(),
                'Price' => $product->getPrice(),
                'Category' => $product->getCategory()
            ];
        }

        return $data;
    }

    /**
     * @throws InvalidOptionsException
     *
     * @return \Symfony\Component\Form\FormView
     */
    public function searchProductsByName()
    {
        $request = $this->request->getCurrentRequest();
        $form = $this->form->create(SearchProductsType::class);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            // TODO
        }

        return $form->createView();
    }
}
```

Here, we simply return the result of our query, this way, we add all
the results into an array and we bind every attribute to a key, we can
use the serializer but the array approach is faster.

Let's update our controller :

```php
<?php

/*
 * This file is part of the SymfonyVue project.
 *
 * (c) Guillaume Loulier <contact@guillaumeloulier.fr>
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 */

namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\OptionsResolver\Exception\InvalidOptionsException;

/**
 * Class ProductsController
 *
 * @author Guillaume Loulier <contact@guillaumeloulier.fr>
 */
class ProductsController extends Controller
{
    /**
     * @Route(path="/products", name="products_all")
     *
     * @throws InvalidOptionsException      Thrown by the form.
     */
    public function getAllProductsAction()
    {
        $products = $this->get('core.products_manager')->getAllProducts();

        $form = $this->get('core.products_manager')->searchProductsByName();

        return $this->render('Logic/products.html.twig', [
            'products' => $products,
            'searchForm' => $form
        ]);
    }

    /**
     * @Route(path="/products/{name}", name="products_all_api")
     *
     * @param string $name      The category of the products
     *
     * @return JsonResponse     The response.
     */
    public function getProductsByNameAction(string $name)
    {
        $data = $this->get('core.products_manager')->getProductsByName($name);

        return new JsonResponse($data, Response::HTTP_OK);
    }
}
```

Here, we simply add a new method with a new route
who require a attributes, this way, we can pass the attribute
to the manager method and return the results via a JSONResponse.



