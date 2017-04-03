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
Once this is done, we define what variables we gonna show inside our bloc, here, it's a simple string bu later, you gonna
learn to manager routes and event forms !
If you do everything correctly, your application (launched by ./bin/console s:r) should show a Hello World ! right in the left of your screen.

Alright, so, we just add Vue into our Symfony apps and we can see that you have a thousand questions, let's be clear, we gonna answer all of yours.
First, let's build a context for our apps, in this course, we gonna build a online e-commerce, nothing to difficult, just a simple market process.

## Part II - Building our logic

Ok, now that we have our logic and even our goals, let's build something solid.