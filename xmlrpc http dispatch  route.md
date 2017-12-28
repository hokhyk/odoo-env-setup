def wsgi_xmlrpc(environ, start_response):
    """ Two routes are available for XML-RPC

    /xmlrpc/<service> route returns faultCode as strings. This is a historic
    violation of the protocol kept for compatibility.

    /xmlrpc/2/<service> is a new route that returns faultCode as int and is
    therefore fully compliant.
    """
    if environ['REQUEST_METHOD'] == 'POST' and environ['PATH_INFO'].startswith('/xmlrpc/'):
        length = int(environ['CONTENT_LENGTH'])
        data = environ['wsgi.input'].read(length)

        # Distinguish betweed the 2 faultCode modes
        string_faultcode = True
        if environ['PATH_INFO'].startswith('/xmlrpc/2/'):
            service = environ['PATH_INFO'][len('/xmlrpc/2/'):]
            string_faultcode = False
        else:
            service = environ['PATH_INFO'][len('/xmlrpc/'):]

        params, method = xmlrpclib.loads(data)
        return xmlrpc_return(start_response, service, method, params, string_faultcode)





def application_unproxied(environ, start_response):
    """ WSGI entry point."""
    # cleanup db/uid trackers - they're set at HTTP dispatch in
    # web.session.OpenERPSession.send() and at RPC dispatch in
    # odoo.service.web_services.objects_proxy.dispatch().
    # /!\ The cleanup cannot be done at the end of this `application`
    # method because werkzeug still produces relevant logging afterwards 
    if hasattr(threading.current_thread(), 'uid'):
        del threading.current_thread().uid
    if hasattr(threading.current_thread(), 'dbname'):
        del threading.current_thread().dbname

    with odoo.api.Environment.manage():
        # Try all handlers until one returns some result (i.e. not None).
        for handler in [wsgi_xmlrpc, odoo.http.root]:
            result = handler(environ, start_response)
            if result is None:
                continue
            return result

    # We never returned from the loop.
    response = 'No handler found.\n'
    start_response('404 Not Found', [('Content-Type', 'text/plain'), ('Content-Length', str(len(response)))])
    return [response]




def application(environ, start_response):
    if config['proxy_mode'] and 'HTTP_X_FORWARDED_HOST' in environ:
        return werkzeug.contrib.fixers.ProxyFix(application_unproxied)(environ, start_response)
    else:
        return application_unproxied(environ, start_response)




