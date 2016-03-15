# OPAL Discoverable Features

OPAL makes extensive use of the case of discoverable features. These are pieces of
composable functionality that can be implemented by any plugin or application,
simply by declaring a class with the appropriate fields and methods implemented. (Details
of exactly which fields and methods vary by feature.)

### Defining Discoverable Features

The only thing required for a discoverable feature, is that it defines the name of
the module in which it should expect to find instances.

    from opal.core import discoverable

    class MyFeature(discoverable.DiscoverableFeature):
        module_name = 'myfeature'


This now gives us acces to various interfaces which come with the library. We can access
all subclasses of `MyFeature` by calling `MyFeature.list()` - which will look through all
Django apps in our application, and look for a module named `myfeature.py`. The `list()`
method then returns any subclasses of `MyFeature`.

### Slugs and Getting Subclasses

Once we define a `name` attribute on sublasses of our feature, we can then fetch them via
the `get()` api.

    class RedFeature(MyFeature): pass

    MyFeature.get('red_feature') # -> RedFeature


### Sortable Features

We can make our feature sortable via an `order` property by including
 `discoverable.SortableFeature` as a parent class.

     class MyFeature(discoverable.DiscoverableFeature, discoverable.SortableFeature):
         module_name = 'myfeature'

This will ensure that `MyFeature.list()` respects the `.order` number of any subclass.

### Restrictable Features

We can ensure that only particular users can access a feature by including
 `discoverable.RestrictableFeature` as a parent class, and implementing the `visible_to`
 classmethod on any restricted subclasses.

     class MyFeature(discoverable.DiscoverableFeature, discoverable.RestrictableFeature):
         module_name = 'myfeature'

 For instance, a feature that was only visible to superusers could be implemented like this:

     class SuperuserFeature(MyFeature):

         @classmethod
         def visible_to(klass, user):
             return user.is_superuser

### Validating Features

Sometimes we wish to validate features so that we don't cause unintended consequences when
we implement subclasses of them. This is available via the `is_valid` classmethod. For instance,
if we wanted to implement a "Bomb" feature, which blew up every time the blow_up attribute was
true, we could to this as follows:

    class BombFeature(discoverable.DiscoverableFeature):
        module_name = 'bombs'
        blow_up = False

        @classmethod
        def is_valid(klass):
            if klass.blow_up == True:
                from opal.core.exceptions import InvalidDiscoverableFeatureError
                raise InvalidDiscoverableFeatureError('BLOWING UP')


    class Threat(BombFeature): pass
    # That's fine.

    class Detonate(BombFeature):
        blow_up = True
    # InvalidDiscoverableFeatureError: BLOWING UP