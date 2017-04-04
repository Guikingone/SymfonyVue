Hi everyone, welcome to this completely new approach of developing web applications (well, in fact, let's use the word "reactive app") using Symfony.

During this "course", we gonna show you how to integrate, use and optimize VueJS inside of Symfony, this way,
your application can be improved and have a better time for user experience.

Alright, time to discover why this course, the idea behind this course is to help you understand why Symfony + VueJS is the best way to build
'future proof' applications, in fact, it can seems strange but Vue is already used by Laravel and the way you can improve things
in this type of architecture is pretty amazing, things says, Symfony NEED to be improved and we're proud of being
contributors to the documentation of this amazing framework but sometimes, the infos that we need is not on the doc's
or not completely explained and we lost a huge amount of time into research and research for nothing.
In this vision, we gonna try to merge parts of this course on the Symfony doc and help you use the framework with the latest
tools available.

Alright, enough talks, let's build our first page with Vue and Symfony !

# Part I - You say Vue ?

Alright, so, what's VueJS ? In fact, Vue (pronounced view) is a frontend framework, a JS framework, yeah,
we know, a JS framework inside a PHP framework, can seems a little bit overwhelming but that's the future
of development and in fact, it can make you better developer over time.

That's say, Vue is a lightweight framework and like other JS framework, his goal is to help you manage
the frontend part of your application and improve the 'reactivity' of this part, talking about reactivity,
a Symfony app is already capable of managing some part of the frontend experience but Vue allow us to go
further and deeper into the user experience.
For more informations, we let you read the official documentation of Vue, this course is not intended to teach you all
the parts of this huge framework.

Ok, time to build something !

## I - Building our skeleton

Alright, let's build our application, first, Symfony installation :

```bash
composer create-project symfony/framework-standard-edition
```

Once the installation is over, the structure should not scare you, we gonna update the DefaultController first :

```php
<?php

namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;

class DefaultController extends Controller
{
    /**
     * @Route("/", name="homepage")
     */
    public function indexAction(Request $request)
    {
        // replace this example code with whatever you need
        return $this->render('default/index.html.twig');
    }
}
```

Now, time to include VueJS in our application, for this, let's update the base.html.twig file :

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
        {% endblock %}
    </body>
</html>
```

Ok, here, we load the Vue library from unpkg, this help to improve the speed of loading the library but let's be clear,
we gonna see later how to 'include' a complete 'manageable' installation of Vue via Webpack.
Once this is done, let's update our index.html.twig file :

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% verbatim %}
      <h1>{{ message }}</h1>
    {% endverbatim %}
{% endblock %}

{% block javascripts %}
    {{ parent() }}
    <script>
        var app = new Vue({
            el: "#app",
            data: {
                message: 'Hello World !'
            }
        })
    </script>
{% endblock %}
```

Here, the first things is to use the verbatim tag via Twig, this way, our JS code is managed by Vue and the syntax
can't be managed by Twig (with the holy magic error 'message' variable can't be found), Vue use the same syntax as Twig (without some shortcut),
this way, we can define our logic and use it faster and smoother.

Second things, we instantiate the Vue object, this way, we can manage our whole page, to do this, we add a id with the value "app"
inside the html body, this way, the body bloc defined by Twig is completely managed by Vue and we can access to the "bloc"
by using the el key, we target the id and voila.

Once this is done, we define what variables we gonna show inside our bloc, here, it's a simple string but later, you gonna
learn to manager routes and even forms !
If you do everything correctly, your application (launched by ./bin/console s:r) should show a 'Hello World !' right in the left of your screen.

Alright, so, we just add Vue into our Symfony apps and we can see that you have a thousand questions, let's be clear, we gonna answer all of yours.
First, let's build a context for our apps, in this course, we gonna build a online e-commerce, nothing to difficult, just a simple market process.

## Part II - Building our logic

Ok, now that we have our logic and even our goals, let's build something solid.

Long story short, let's add something to render to the client, in a e-commerce shop, we have multiples 'products', in this
vision, we gonna build a Product entity and the manager who manage this entity :

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
}
```

Simple entity, just for the "gestion" purpose, this way, here's the manager :

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
use Doctrine\ORM\EntityManager;

/**
 * Class ProductsManager
 *
 * @author Guillaume Loulier <contact@guillaumeloulier.fr>
 */
class ProductsManager
{
    /** @var EntityManager */
    private $doctrine;

    /**
     * ProductsManager constructor.
     *
     * @param EntityManager $doctrine
     */
    public function __construct (EntityManager $doctrine)
    {
        $this->doctrine = $doctrine;
    }

    /**
     * @return array
     */
    public function getAllProducts()
    {
        return $this->doctrine->getRepository(Products::class)->findAll();
    }
}
```

Here, nothing to hard, we just return all the products stored into the BDD (for the moment).

Alright, let's update our Controllers :

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

class ProductsController extends Controller
{
    /**
     * @Route(path="/products", name="products_all")
     */
    public function getAllProductsAction()
    {
        $products = $this->get('core.products_manager')->getAllProducts();

        return $this->render('Logic/products.html.twig', [
            'products' => $products
        ]);
    }
}
```

The core.products_manager is declared inside the services.yml :

```yaml
# Learn more about services, parameters and containers at
# http://symfony.com/doc/current/service_container.html
parameters:
    #parameter_name: value

services:
    core.products_manager:
        class: AppBundle\Managers\ProductsManager
        arguments:
            - '@doctrine.orm.entity_manager'
```

For the moment, not a single products has been persisted into the BDD but we already have our logic, nice things.
For the moment, we have a routes who return all the products, good idea but how can we access her from Vue ?
Well, if you use Twig, you probably use something like this :

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% verbatim %}
      <h1>{{ message }}</h1>
    {% endverbatim %}
    <a href="{{ path('products_all') }}">Products !</a>
{% endblock %}

{% block javascripts %}
    {{ parent() }}
    <script>
        var app = new Vue({
            el: "#app",
            data: {
                message: 'Hello World !',
                link: 'products'
            }
        })
    </script>
{% endblock %}
```

Don't get me wrong, that's a good way for approaching routing but Vue can be used in order to do the same things but
faster and cleaner, in fact, Vue has his own router, we gonna use it later but for the moment, let's see how to create a
link with Vue from the index to the /products route.

To do so, let's update our index.html.twig file :

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% verbatim %}
      <h1>{{ message }}</h1>
      <p>
        <a v-bind:href="link">Products !</a>
      </p>
    {% endverbatim %}
{% endblock %}

{% block javascripts %}
    {{ parent() }}
    <script>
        var app = new Vue({
            el: "#app",
            data: {
                message: 'Hello World !',
                link: 'products'
            }
        })
    </script>
{% endblock %}
```

Alright, so, where's the differences here ?

First, we define a new attribute in the Vue instance, we simply call the after slash part of our route, if we need
to have a slash after, Vue can handle it. Once this is done, time to "bind" the value into our view, to do this, Vue
allow to use a new "html attribute" called v-bind, this attribute allow to show in the raw aspect a attribute and if it's
a URL attribute, to passe it into the URL, amazing little thing.
In this case, we bind into the href part of the a tag, we simply passe the attribute and Vue gonna do the trick to pass
the attribute to the URL, once this is done, Symfony gonna grab the request and match with the controller created earlier,
yeah, we know, magic thing, amazing approach, we know, we love this type of logic !

If everything goes right, you should see a blank page with no error, logic approach, we don't have any products at this time.

Alright, time to build something bigger, we know how to show something, we know how to merge the Sf router and the Vue instance,
let's build the next part of our application.

## Part III - You say logic ?

Alright random citizen, time to get serious, now that we have some logic, time to build something bigger.
In this part, we gonna discover something bigger, for the moment, we have a backend and some frontend logic, time
to connect the two and build something really smarter.

For the moment, Vue only handle basic usage, we've connect the "router" aspect but later, we gonna pass through this one,
so, what's the deal here ?

The biggest problem is that we use Vue as a "magical" tools in order to show something, not really what Vue is designed for,
in fact, Vue is build in order to manage the user experience and build a complete Single Page Application, this way,
our backend only return "Json content" and Vue provide the view aspect and the user management part of the application.

This way, if we go deeper, we need to have some "real" logic that Vue can handle.

Alright, so, what can be build using Vue ?

In our application, we can imagine a research form who can handle the user demand and show the result found on the BDD
into a list or better, show a list of result and allow the user to click on the results and go to the details page.
Yeah, that's what we want, time to build it !

First, let's create a SearchForm and a method inside our Manager to handle this form :

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

namespace AppBundle\Form\Type;

use AppBundle\Entity\Products;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class SearchProductsType extends AbstractType
{
    public function buildForm (FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('name', TextType::class)
        ;
    }

    public function configureOptions (OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => Products::class
        ]);
    }

}
```

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

Ok, let's update our controllers then :

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

/**
 * Class ProductsController
 *
 * @author Guillaume Loulier <contact@guillaumeloulier.fr>
 */
class ProductsController extends Controller
{
    /**
     * @Route(path="/products", name="products_all")
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
}
```

Alright, time to get serious, you're about to discover something completely new in web development, we named it
'mind changer components approach' but well, way to long to say, call it components.
You need to understand basic logic to understand component, in the past, when you build a HTML page, you create new tag
and "cut" your page in multiples sections or article and event div, bad idea.

Today, you cut your page on multiples component, each components contains his own logic and is "outside" of the other component,
this approach is the heart of React and Angular, in Vue, it's also the heart but the approach is much simple, in fact,
you can build the whole application without using component, if you use components, Vue's gonna be a completely different
framework and your code gonna change fast and for the best !

Ok, enough talk, how can we build component in our view ? Simply by extending the Vue instance :

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% for products in products %}
        <p>{{ products.name }}</p>
    {% endfor %}
    <products-component></products-component>
{% endblock %}

{% block javascripts %}
    {{ parent() }}
    <script>
        Vue.component('products-component', {
            template: '<div>Hello World from products !</div>'
        });
        var app = new Vue({
            el: '#app',
            data: {

            },
        })
    </script>
{% endblock %}
```

Here's your first component using Vue !

if everything goes right, you must see 'Hello World from products !' in your browser, petty cool huh ?
In fact, Vue is based on Polymer approach, by the way that you can build your own html tag and 'component' as long
as he contains logic and can be called in your view, pretty simple.

Ok, now, time to update our view in order to show the form :

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% for products in products %}
        <p>{{ products.name }}</p>
    {% endfor %}
    {{ form_start(searchForm) }}
    {{ form_label(searchForm.name) }}
        {{ form_widget(searchForm.name) }}
        {{ form_errors(searchForm.name) }}
    {{ form_end(searchForm) }}
    <products-component></products-component>
{% endblock %}

{% block javascripts %}
    {{ parent() }}
    <script>
        Vue.component('products-component', {
            template: '<div>Hello World from products !</div>'
        });
        var app = new Vue({
            el: '#app',
            data: {

            },
        })
    </script>
{% endblock %}
```

Alright, simple things here, we simply show how to display the form, now, time to get serious, how can we use Vue inside
of our form ?

Hum, not so easy if you think about it ... In fact, it's very simple, we learn earlier how to bind a attribute to the URL,
magic happen, Vue is also capable of binding attributes FROM something, like a form for example, let's update our view :

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% for products in products %}
        <p>{{ products.name }}</p>
    {% endfor %}
    {{ form_start(searchForm) }}
    {{ form_label(searchForm.name) }}
        {{ form_widget(searchForm.name, {'attr': {'v-model': 'name'}}) }}
        {{ form_errors(searchForm.name) }}
    {{ form_end(searchForm) }}
    {% verbatim %}
        <p>
            How, seems like you're searching {{ name }},
            here's the list of products wih this category that we have in stock :
        </p>
    {% endverbatim %}
{% endblock %}

{% block javascripts %}
    {{ parent() }}
    <script>
        Vue.component('products-component', {
            template: '<div>How, seems like you\'re searching </div>'
        });
        var app = new Vue({
            el: '#app',
            data: {
                name: ''
            },
        })
    </script>
{% endblock %}
```

Here's the big advantages, here, we delete the component for readability but don't be scare, he's on the return,
for the logic, we simply put the v-model attribute on our form, this way, Vue can bind this attribute from our form to
our Vue instance and store the result, once this is done, we show the result in our view via {{ name }}, just give a try,
if you type 'illumination !' inside the form, Vue's gonna show the result in the p tag, we can even go further with
a conditioning rendering :

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% for products in products %}
        <p>{{ products.name }}</p>
    {% endfor %}
    {{ form_start(searchForm) }}
    {{ form_label(searchForm.name) }}
        {{ form_widget(searchForm.name, {'attr': {'v-model': 'name'}}) }}
        {{ form_errors(searchForm.name) }}
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
        Vue.component('products-component', {
            template: '<div>Hello World from products !</div>'
        });
        var app = new Vue({
            el: '#app',
            data: {
                name: ''
            },
        })
    </script>
{% endblock %}
```

Here, we simply add the v-if attribute on the p with the condition on "if name exist then the p tag gonna be updated"
with the content of the form and displayed to the visitor.

Yeah, we know, seems magic ? It is ! This way, if the form stay blank, the visitor don't have the p and he can continue
to explore the page, if he enter something, the p is updated and we show the result, later, we gonna ask the server for
the results and show this result to the client but for the moment, give a try by handling multiples form
and binding the attributes in your view.

Ok, one more things, here, we sam how to bind the value passed from the input field into our Vue instance
and how to return this attributes into our view but how can we 'inject' Vue into the Submit process of Symfony and grab
 the data, this way, we can stop the process of the form and simply launch the research phase from Vue ?

Well, for this, Vue allow to use the 'v-on' directives, this approach allow to call Vue on certain phase or events
and do things related to this event. In this way, we can call the submit phase of the form and do something else than
process the submission via Symfony :

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
        Vue.component('products-component', {
            template: '<div>Hello World from products !</div>'
        });
        var app = new Vue({
            el: '#app',
            data: {
                name: ''
            },
            methods: {
                searchProducts: function() {
                    console.log('Hey, you\'re searching ' +  this.name);
                }
            }
        })
    </script>
{% endblock %}
```

Here, we've add the v-on:submit.prevent directive on the form_start call and bind this directive to a method called
searchProducts, this way, Vue's gonna do the relation and call the method once the form is submitted, we gonna stop the
the submit process and log to the console the message, if everything goes smoothly, your console should show
the message.

Hell yeah ! We know, Vue is awesome, fast and simple to understand, his approach to a lot of things is pretty straight forward
but let's be clear, you don't see the whole package that Vue offer, here, we simply inject Vue inside our view and call
different method or shortcut in order to do simple things, that's not the complete way of using Vue.

Alright crazy readers, here's just the beginning of your adventure with Vue and Symfony, it's over for this chapter but
we gonna build something way more bigger in the second chapter, stay in contact !

Guillaume, brownie addict.




