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
For the moment, we have a routes who return all the products, good ide but how can we access her from Vue ?
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

If everything go's right, you should see a black page with no error, logic approach, we don't have any products at this time.

Alright, time to build something bigger, we know how to show something, we know how to merge the Sf router and the Vue instance,
let's build the next part of our application.

## Part III - You say logic ?