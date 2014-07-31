Title: Simple Nagios
Date: 2012-02-01 10:20
Category: blog

A quick look at a read-only web view of nagios, called 'simple nagios'.

This was written to allow the viewing of nagios as simple as possible, and the customizing of the nagios views as simple as possible.

This has been achieved by splitting the generation of the data from the presentation of the data using Mk_livestatus to gather the data from nagios.
By using flask and jinja2 templates and twitters bootstrap to present the data to the end user.


The request code in side of flask ends up nice and clean

```python
@App.route("/")
@App.route("/tac/")
def tac():
    """
    The 'Tactical Monitoring Overview', this will show an overview of all 
    the services: up, down, warning, error, etc.
    """
    # Generate the extra filters based on query strings
    extra_filters = gather_filters(request)
    # Grab all the service stats
    service_stats = query.service_stats(extra_filter=extra_filters)
    # Grab all the host stats
    host_stats = query.hosts_stats(extra_filter=extra_filters)
    # Render the template.
    return render_template('tac.template', service_stats=service_stats, 
                           host_stats=host_stats )
```

Here is an extract of some of the template code used to generate the pages.

```html
 <div class='span5'>
    <h3> {{host.groups|length}} Host Groups </h3>
    <p>
    Groups that {{host.name}} is a member of.<Br />
    clicking a host group will give you a list of hosts that are in that group.
    <p>
    {% for host_group in host.groups %}
         <a href='/hostgroup/{{host_group}}/' title='Show the status of all hosts beloing to {{host_group}}' >{{host_group}}</a>,
    {% endfor %}
     </p>
 </div>
```

The project can be found over on git-hub [daniellawrence/simplenagios](https://github.com/daniellawrence/simplenagios).

screen shots
------------

This is a refreshed approach to the Tactical Overview or the tac.cgi.

![TAC](https://raw.github.com/daniellawrence/simplenagios/master/screenshots/tac.png "TAC")
