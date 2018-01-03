http://werkzeug.pocoo.org/docs/0.12/routing/

import werkzeug.contrib.sessions
import werkzeug.datastructures
import werkzeug.exceptions
import werkzeug.local
import werkzeug.routing
import werkzeug.wrappers
import werkzeug.wsgi
from werkzeug.wsgi import wrap_file

URL Routing

When it comes to combining multiple controller or view functions (however you want to call them), you need a dispatcher. A simple way would be applying regular expression tests on PATH_INFO and call registered callback functions that return the value.

Werkzeug provides a much more powerful system, similar to Routes. All the objects mentioned on this page must be imported from werkzeug.routing, not from werkzeug!
Quickstart

Here is a simple example which could be the URL definition for a blog:

from werkzeug.routing import Map, Rule, NotFound, RequestRedirect

url_map = Map([
    Rule('/', endpoint='blog/index'),
    Rule('/<int:year>/', endpoint='blog/archive'),
    Rule('/<int:year>/<int:month>/', endpoint='blog/archive'),
    Rule('/<int:year>/<int:month>/<int:day>/', endpoint='blog/archive'),
    Rule('/<int:year>/<int:month>/<int:day>/<slug>',
         endpoint='blog/show_post'),
    Rule('/about', endpoint='blog/about_me'),
    Rule('/feeds/', endpoint='blog/feeds'),
    Rule('/feeds/<feed_name>.rss', endpoint='blog/show_feed')
])

def application(environ, start_response):
    urls = url_map.bind_to_environ(environ)
    try:
        endpoint, args = urls.match()
    except HTTPException, e:
        return e(environ, start_response)
    start_response('200 OK', [('Content-Type', 'text/plain')])
    return ['Rule points to %r with arguments %r' % (endpoint, args)]

So what does that do? First of all we create a new Map which stores a bunch of URL rules. Then we pass it a list of Rule objects.

Each Rule object is instantiated with a string that represents a rule and an endpoint which will be the alias for what view the rule represents. Multiple rules can have the same endpoint, but should have different arguments to allow URL construction.

The format for the URL rules is straightforward, but explained in detail below.

Inside the WSGI application we bind the url_map to the current request which will return a new MapAdapter. This url_map adapter can then be used to match or build domains for the current request.

The MapAdapter.match() method can then either return a tuple in the form (endpoint, args) or raise one of the three exceptions NotFound, MethodNotAllowed, or RequestRedirect. For more details about those exceptions have a look at the documentation of the MapAdapter.match() method.
Rule Format

Rule strings basically are just normal URL paths with placeholders in the format <converter(arguments):name>, where converter and the arguments are optional. If no converter is defined, the default converter is used (which means string in the normal configuration).

URL rules that end with a slash are branch URLs, others are leaves. If you have strict_slashes enabled (which is the default), all branch URLs that are visited without a trailing slash will trigger a redirect to the same URL with that slash appended.

The list of converters can be extended, the default converters are explained below.
Builtin Converters

Here a list of converters that come with Werkzeug:

class werkzeug.routing.UnicodeConverter(map, minlength=1, maxlength=None, length=None)

    This converter is the default converter and accepts any string but only one path segment. Thus the string can not include a slash.

    This is the default validator.

    Example:

    Rule('/pages/<page>'),
    Rule('/<string(length=2):lang_code>')

    Parameters:	

        map – the Map.
        minlength – the minimum length of the string. Must be greater or equal 1.
        maxlength – the maximum length of the string.
        length – the exact length of the string.

class werkzeug.routing.PathConverter(map)

    Like the default UnicodeConverter, but it also matches slashes. This is useful for wikis and similar applications:

    Rule('/<path:wikipage>')
    Rule('/<path:wikipage>/edit')

    Parameters:	map – the Map.

class werkzeug.routing.AnyConverter(map, *items)

    Matches one of the items provided. Items can either be Python identifiers or strings:

    Rule('/<any(about, help, imprint, class, "foo,bar"):page_name>')

    Parameters:	

        map – the Map.
        items – this function accepts the possible items as positional arguments.

class werkzeug.routing.IntegerConverter(map, fixed_digits=0, min=None, max=None)

    This converter only accepts integer values:

    Rule('/page/<int:page>')

    This converter does not support negative values.
    Parameters:	

        map – the Map.
        fixed_digits – the number of fixed digits in the URL. If you set this to 4 for example, the application will only match if the url looks like /0001/. The default is variable length.
        min – the minimal value.
        max – the maximal value.

class werkzeug.routing.FloatConverter(map, min=None, max=None)

    This converter only accepts floating point values:

    Rule('/probability/<float:probability>')

    This converter does not support negative values.
    Parameters:	

        map – the Map.
        min – the minimal value.
        max – the maximal value.

class werkzeug.routing.UUIDConverter(map)

    This converter only accepts UUID strings:

    Rule('/object/<uuid:identifier>')

    New in version 0.10.
    Parameters:	map – the Map.

Maps, Rules and Adapters

class werkzeug.routing.Map(rules=None, default_subdomain=”, charset=’utf-8’, strict_slashes=True, redirect_defaults=True, converters=None, sort_parameters=False, sort_key=None, encoding_errors=’replace’, host_matching=False)

    The map class stores all the URL rules and some configuration parameters. Some of the configuration values are only stored on the Map instance since those affect all rules, others are just defaults and can be overridden for each rule. Note that you have to specify all arguments besides the rules as keyword arguments!
    Parameters:	

        rules – sequence of url rules for this map.
        default_subdomain – The default subdomain for rules without a subdomain defined.
        charset – charset of the url. defaults to "utf-8"
        strict_slashes – Take care of trailing slashes.
        redirect_defaults – This will redirect to the default rule if it wasn’t visited that way. This helps creating unique URLs.
        converters – A dict of converters that adds additional converters to the list of converters. If you redefine one converter this will override the original one.
        sort_parameters – If set to True the url parameters are sorted. See url_encode for more details.
        sort_key – The sort key function for url_encode.
        encoding_errors – the error method to use for decoding
        host_matching – if set to True it enables the host matching feature and disables the subdomain one. If enabled the host parameter to rules is used instead of the subdomain one.

    New in version 0.5: sort_parameters and sort_key was added.

    New in version 0.7: encoding_errors and host_matching was added.

    converters

        The dictionary of converters. This can be modified after the class was created, but will only affect rules added after the modification. If the rules are defined with the list passed to the class, the converters parameter to the constructor has to be used instead.

    add(rulefactory)

        Add a new rule or factory to the map and bind it. Requires that the rule is not bound to another map.
        Parameters:	rulefactory – a Rule or RuleFactory

    bind(server_name, script_name=None, subdomain=None, url_scheme=’http’, default_method=’GET’, path_info=None, query_args=None)

        Return a new MapAdapter with the details specified to the call. Note that script_name will default to '/' if not further specified or None. The server_name at least is a requirement because the HTTP RFC requires absolute URLs for redirects and so all redirect exceptions raised by Werkzeug will contain the full canonical URL.

        If no path_info is passed to match() it will use the default path info passed to bind. While this doesn’t really make sense for manual bind calls, it’s useful if you bind a map to a WSGI environment which already contains the path info.

        subdomain will default to the default_subdomain for this map if no defined. If there is no default_subdomain you cannot use the subdomain feature.

        New in version 0.7: query_args added

        New in version 0.8: query_args can now also be a string.

    bind_to_environ(environ, server_name=None, subdomain=None)

        Like bind() but you can pass it an WSGI environment and it will fetch the information from that dictionary. Note that because of limitations in the protocol there is no way to get the current subdomain and real server_name from the environment. If you don’t provide it, Werkzeug will use SERVER_NAME and SERVER_PORT (or HTTP_HOST if provided) as used server_name with disabled subdomain feature.

        If subdomain is None but an environment and a server name is provided it will calculate the current subdomain automatically. Example: server_name is 'example.com' and the SERVER_NAME in the wsgi environ is 'staging.dev.example.com' the calculated subdomain will be 'staging.dev'.

        If the object passed as environ has an environ attribute, the value of this attribute is used instead. This allows you to pass request objects. Additionally PATH_INFO added as a default of the MapAdapter so that you don’t have to pass the path info to the match method.

        Changed in version 0.5: previously this method accepted a bogus calculate_subdomain parameter that did not have any effect. It was removed because of that.

        Changed in version 0.8: This will no longer raise a ValueError when an unexpected server name was passed.
        Parameters:	

            environ – a WSGI environment.
            server_name – an optional server name hint (see above).
            subdomain – optionally the current subdomain (see above).

    default_converters = ImmutableDict({‘int’: <class ‘werkzeug.routing.IntegerConverter’>, ‘string’: <class ‘werkzeug.routing.UnicodeConverter’>, ‘default’: <class ‘werkzeug.routing.UnicodeConverter’>, ‘path’: <class ‘werkzeug.routing.PathConverter’>, ‘float’: <class ‘werkzeug.routing.FloatConverter’>, ‘any’: <class ‘werkzeug.routing.AnyConverter’>, ‘uuid’: <class ‘werkzeug.routing.UUIDConverter’>})

        New in version 0.6: a dict of default converters to be used.

    is_endpoint_expecting(endpoint, *arguments)

        Iterate over all rules and check if the endpoint expects the arguments provided. This is for example useful if you have some URLs that expect a language code and others that do not and you want to wrap the builder a bit so that the current language code is automatically added if not provided but endpoints expect it.
        Parameters:	

            endpoint – the endpoint to check.
            arguments – this function accepts one or more arguments as positional arguments. Each one of them is checked.

    iter_rules(endpoint=None)

        Iterate over all rules or the rules of an endpoint.
        Parameters:	endpoint – if provided only the rules for that endpoint are returned.
        Returns:	an iterator

    update()

        Called before matching and building to keep the compiled rules in the correct order after things changed.

class werkzeug.routing.MapAdapter(map, server_name, script_name, subdomain, url_scheme, path_info, default_method, query_args=None)

    Returned by Map.bind() or Map.bind_to_environ() and does the URL matching and building based on runtime information.

    allowed_methods(path_info=None)

        Returns the valid methods that match for a given path.

        New in version 0.7.

    build(endpoint, values=None, method=None, force_external=False, append_unknown=True)

        Building URLs works pretty much the other way round. Instead of match you call build and pass it the endpoint and a dict of arguments for the placeholders.

        The build function also accepts an argument called force_external which, if you set it to True will force external URLs. Per default external URLs (include the server name) will only be used if the target URL is on a different subdomain.

        >>> m = Map([
        ...     Rule('/', endpoint='index'),
        ...     Rule('/downloads/', endpoint='downloads/index'),
        ...     Rule('/downloads/<int:id>', endpoint='downloads/show')
        ... ])
        >>> urls = m.bind("example.com", "/")
        >>> urls.build("index", {})
        '/'
        >>> urls.build("downloads/show", {'id': 42})
        '/downloads/42'
        >>> urls.build("downloads/show", {'id': 42}, force_external=True)
        'http://example.com/downloads/42'

        Because URLs cannot contain non ASCII data you will always get bytestrings back. Non ASCII characters are urlencoded with the charset defined on the map instance.

        Additional values are converted to unicode and appended to the URL as URL querystring parameters:

        >>> urls.build("index", {'q': 'My Searchstring'})
        '/?q=My+Searchstring'

        When processing those additional values, lists are furthermore interpreted as multiple values (as per werkzeug.datastructures.MultiDict):

        >>> urls.build("index", {'q': ['a', 'b', 'c']})
        '/?q=a&q=b&q=c'

        If a rule does not exist when building a BuildError exception is raised.

        The build method accepts an argument called method which allows you to specify the method you want to have an URL built for if you have different methods for the same endpoint specified.

        New in version 0.6: the append_unknown parameter was added.
        Parameters:	

            endpoint – the endpoint of the URL to build.
            values – the values for the URL to build. Unhandled values are appended to the URL as query parameters.
            method – the HTTP method for the rule if there are different URLs for different methods on the same endpoint.
            force_external – enforce full canonical external URLs. If the URL scheme is not provided, this will generate a protocol-relative URL.
            append_unknown – unknown parameters are appended to the generated URL as query string argument. Disable this if you want the builder to ignore those.

    dispatch(view_func, path_info=None, method=None, catch_http_exceptions=False)

        Does the complete dispatching process. view_func is called with the endpoint and a dict with the values for the view. It should look up the view function, call it, and return a response object or WSGI application. http exceptions are not caught by default so that applications can display nicer error messages by just catching them by hand. If you want to stick with the default error messages you can pass it catch_http_exceptions=True and it will catch the http exceptions.

        Here a small example for the dispatch usage:

        from werkzeug.wrappers import Request, Response
        from werkzeug.wsgi import responder
        from werkzeug.routing import Map, Rule

        def on_index(request):
            return Response('Hello from the index')

        url_map = Map([Rule('/', endpoint='index')])
        views = {'index': on_index}

        @responder
        def application(environ, start_response):
            request = Request(environ)
            urls = url_map.bind_to_environ(environ)
            return urls.dispatch(lambda e, v: views[e](request, **v),
                                 catch_http_exceptions=True)

        Keep in mind that this method might return exception objects, too, so use Response.force_type to get a response object.
        Parameters:	

            view_func – a function that is called with the endpoint as first argument and the value dict as second. Has to dispatch to the actual view function with this information. (see above)
            path_info – the path info to use for matching. Overrides the path info specified on binding.
            method – the HTTP method used for matching. Overrides the method specified on binding.
            catch_http_exceptions – set to True to catch any of the werkzeug HTTPExceptions.

    get_default_redirect(rule, method, values, query_args)

        A helper that returns the URL to redirect to if it finds one. This is used for default redirecting only.
        Internal:	

    get_host(domain_part)

        Figures out the full host name for the given domain part. The domain part is a subdomain in case host matching is disabled or a full host name.

    make_alias_redirect_url(path, endpoint, values, method, query_args)

        Internally called to make an alias redirect URL.

    make_redirect_url(path_info, query_args=None, domain_part=None)

        Creates a redirect URL.
        Internal:	

    match(path_info=None, method=None, return_rule=False, query_args=None)

        The usage is simple: you just pass the match method the current path info as well as the method (which defaults to GET). The following things can then happen:

            you receive a NotFound exception that indicates that no URL is matching. A NotFound exception is also a WSGI application you can call to get a default page not found page (happens to be the same object as werkzeug.exceptions.NotFound)
            you receive a MethodNotAllowed exception that indicates that there is a match for this URL but not for the current request method. This is useful for RESTful applications.
            you receive a RequestRedirect exception with a new_url attribute. This exception is used to notify you about a request Werkzeug requests from your WSGI application. This is for example the case if you request /foo although the correct URL is /foo/ You can use the RequestRedirect instance as response-like object similar to all other subclasses of HTTPException.
            you get a tuple in the form (endpoint, arguments) if there is a match (unless return_rule is True, in which case you get a tuple in the form (rule, arguments))

        If the path info is not passed to the match method the default path info of the map is used (defaults to the root URL if not defined explicitly).

        All of the exceptions raised are subclasses of HTTPException so they can be used as WSGI responses. The will all render generic error or redirect pages.

        Here is a small example for matching:

        >>> m = Map([
        ...     Rule('/', endpoint='index'),
        ...     Rule('/downloads/', endpoint='downloads/index'),
        ...     Rule('/downloads/<int:id>', endpoint='downloads/show')
        ... ])
        >>> urls = m.bind("example.com", "/")
        >>> urls.match("/", "GET")
        ('index', {})
        >>> urls.match("/downloads/42")
        ('downloads/show', {'id': 42})

        And here is what happens on redirect and missing URLs:

        >>> urls.match("/downloads")
        Traceback (most recent call last):
          ...
        RequestRedirect: http://example.com/downloads/
        >>> urls.match("/missing")
        Traceback (most recent call last):
          ...
        NotFound: 404 Not Found

        Parameters:	

            path_info – the path info to use for matching. Overrides the path info specified on binding.
            method – the HTTP method used for matching. Overrides the method specified on binding.
            return_rule – return the rule that matched instead of just the endpoint (defaults to False).
            query_args – optional query arguments that are used for automatic redirects as string or dictionary. It’s currently not possible to use the query arguments for URL matching.

        New in version 0.6: return_rule was added.

        New in version 0.7: query_args was added.

        Changed in version 0.8: query_args can now also be a string.

    test(path_info=None, method=None)

        Test if a rule would match. Works like match but returns True if the URL matches, or False if it does not exist.
        Parameters:	

            path_info – the path info to use for matching. Overrides the path info specified on binding.
            method – the HTTP method used for matching. Overrides the method specified on binding.

class werkzeug.routing.Rule(string, defaults=None, subdomain=None, methods=None, build_only=False, endpoint=None, strict_slashes=None, redirect_to=None, alias=False, host=None)

    A Rule represents one URL pattern. There are some options for Rule that change the way it behaves and are passed to the Rule constructor. Note that besides the rule-string all arguments must be keyword arguments in order to not break the application on Werkzeug upgrades.

    string

        Rule strings basically are just normal URL paths with placeholders in the format <converter(arguments):name> where the converter and the arguments are optional. If no converter is defined the default converter is used which means string in the normal configuration.

        URL rules that end with a slash are branch URLs, others are leaves. If you have strict_slashes enabled (which is the default), all branch URLs that are matched without a trailing slash will trigger a redirect to the same URL with the missing slash appended.

        The converters are defined on the Map.
    endpoint
        The endpoint for this rule. This can be anything. A reference to a function, a string, a number etc. The preferred way is using a string because the endpoint is used for URL generation.
    defaults

        An optional dict with defaults for other rules with the same endpoint. This is a bit tricky but useful if you want to have unique URLs:

        url_map = Map([
            Rule('/all/', defaults={'page': 1}, endpoint='all_entries'),
            Rule('/all/page/<int:page>', endpoint='all_entries')
        ])

        If a user now visits http://example.com/all/page/1 he will be redirected to http://example.com/all/. If redirect_defaults is disabled on the Map instance this will only affect the URL generation.
    subdomain

        The subdomain rule string for this rule. If not specified the rule only matches for the default_subdomain of the map. If the map is not bound to a subdomain this feature is disabled.

        Can be useful if you want to have user profiles on different subdomains and all subdomains are forwarded to your application:

        url_map = Map([
            Rule('/', subdomain='<username>', endpoint='user/homepage'),
            Rule('/stats', subdomain='<username>', endpoint='user/stats')
        ])

    methods

        A sequence of http methods this rule applies to. If not specified, all methods are allowed. For example this can be useful if you want different endpoints for POST and GET. If methods are defined and the path matches but the method matched against is not in this list or in the list of another rule for that path the error raised is of the type MethodNotAllowed rather than NotFound. If GET is present in the list of methods and HEAD is not, HEAD is added automatically.

        Changed in version 0.6.1: HEAD is now automatically added to the methods if GET is present. The reason for this is that existing code often did not work properly in servers not rewriting HEAD to GET automatically and it was not documented how HEAD should be treated. This was considered a bug in Werkzeug because of that.
    strict_slashes
        Override the Map setting for strict_slashes only for this rule. If not specified the Map setting is used.
    build_only
        Set this to True and the rule will never match but will create a URL that can be build. This is useful if you have resources on a subdomain or folder that are not handled by the WSGI application (like static data)
    redirect_to

        If given this must be either a string or callable. In case of a callable it’s called with the url adapter that triggered the match and the values of the URL as keyword arguments and has to return the target for the redirect, otherwise it has to be a string with placeholders in rule syntax:

        def foo_with_slug(adapter, id):
            # ask the database for the slug for the old id.  this of
            # course has nothing to do with werkzeug.
            return 'foo/' + Foo.get_slug_for_id(id)

        url_map = Map([
            Rule('/foo/<slug>', endpoint='foo'),
            Rule('/some/old/url/<slug>', redirect_to='foo/<slug>'),
            Rule('/other/old/url/<int:id>', redirect_to=foo_with_slug)
        ])

        When the rule is matched the routing system will raise a RequestRedirect exception with the target for the redirect.

        Keep in mind that the URL will be joined against the URL root of the script so don’t use a leading slash on the target URL unless you really mean root of that domain.
    alias
        If enabled this rule serves as an alias for another rule with the same endpoint and arguments.
    host
        If provided and the URL map has host matching enabled this can be used to provide a match rule for the whole host. This also means that the subdomain feature is disabled.

    New in version 0.7: The alias and host parameters were added.

    empty()

        Return an unbound copy of this rule.

        This can be useful if want to reuse an already bound URL for another map. See get_empty_kwargs to override what keyword arguments are provided to the new copy.

Rule Factories

class werkzeug.routing.RuleFactory

    As soon as you have more complex URL setups it’s a good idea to use rule factories to avoid repetitive tasks. Some of them are builtin, others can be added by subclassing RuleFactory and overriding get_rules.

    get_rules(map)

        Subclasses of RuleFactory have to override this method and return an iterable of rules.

class werkzeug.routing.Subdomain(subdomain, rules)

    All URLs provided by this factory have the subdomain set to a specific domain. For example if you want to use the subdomain for the current language this can be a good setup:

    url_map = Map([
        Rule('/', endpoint='#select_language'),
        Subdomain('<string(length=2):lang_code>', [
            Rule('/', endpoint='index'),
            Rule('/about', endpoint='about'),
            Rule('/help', endpoint='help')
        ])
    ])

    All the rules except for the '#select_language' endpoint will now listen on a two letter long subdomain that holds the language code for the current request.

class werkzeug.routing.Submount(path, rules)

    Like Subdomain but prefixes the URL rule with a given string:

    url_map = Map([
        Rule('/', endpoint='index'),
        Submount('/blog', [
            Rule('/', endpoint='blog/index'),
            Rule('/entry/<entry_slug>', endpoint='blog/show')
        ])
    ])

    Now the rule 'blog/show' matches /blog/entry/<entry_slug>.

class werkzeug.routing.EndpointPrefix(prefix, rules)

    Prefixes all endpoints (which must be strings for this factory) with another string. This can be useful for sub applications:

    url_map = Map([
        Rule('/', endpoint='index'),
        EndpointPrefix('blog/', [Submount('/blog', [
            Rule('/', endpoint='index'),
            Rule('/entry/<entry_slug>', endpoint='show')
        ])])
    ])

Rule Templates

class werkzeug.routing.RuleTemplate(rules)

    Returns copies of the rules wrapped and expands string templates in the endpoint, rule, defaults or subdomain sections.

    Here a small example for such a rule template:

    from werkzeug.routing import Map, Rule, RuleTemplate

    resource = RuleTemplate([
        Rule('/$name/', endpoint='$name.list'),
        Rule('/$name/<int:id>', endpoint='$name.show')
    ])

    url_map = Map([resource(name='user'), resource(name='page')])

    When a rule template is called the keyword arguments are used to replace the placeholders in all the string parameters.

Custom Converters

You can easily add custom converters. The only thing you have to do is to subclass BaseConverter and pass that new converter to the url_map. A converter has to provide two public methods: to_python and to_url, as well as a member that represents a regular expression. Here is a small example:

from random import randrange
from werkzeug.routing import Rule, Map, BaseConverter, ValidationError

class BooleanConverter(BaseConverter):

    def __init__(self, url_map, randomify=False):
        super(BooleanConverter, self).__init__(url_map)
        self.randomify = randomify
        self.regex = '(?:yes|no|maybe)'

    def to_python(self, value):
        if value == 'maybe':
            if self.randomify:
                return not randrange(2)
            raise ValidationError()
        return value == 'yes'

    def to_url(self, value):
        return value and 'yes' or 'no'

url_map = Map([
    Rule('/vote/<bool:werkzeug_rocks>', endpoint='vote'),
    Rule('/vote/<bool(randomify=True):foo>', endpoint='foo')
], converters={'bool': BooleanConverter})

If you want that converter to be the default converter, name it 'default'.
Host Matching

New in version 0.7.

Starting with Werkzeug 0.7 it’s also possible to do matching on the whole host names instead of just the subdomain. To enable this feature you need to pass host_matching=True to the Map constructor and provide the host argument to all routes:

url_map = Map([
    Rule('/', endpoint='www_index', host='www.example.com'),
    Rule('/', endpoint='help_index', host='help.example.com')
], host_matching=True)

Variable parts are of course also possible in the host section:

url_map = Map([
    Rule('/', endpoint='www_index', host='www.example.com'),
    Rule('/', endpoint='user_index', host='<user>.example.com')
], host_matching=True)


