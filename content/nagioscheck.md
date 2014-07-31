Title: nagioscheck
Date: 2014-07-31 18:20
Category: blog

This is an example of using puppetdb with a nagios server to allow you to run all
your configured nagios checks via puppet on the local machine in question.

Some example output will look like this...

    PLACEHOLDER
    PLACEHOLDER
    PLACEHOLDER

The idea is to let you get instant feedback on your actions on a local machine,
without the need to leave your terminal.

The script will query your puppetdb machine looking for all the nagios checks that
should be applied to the current servername (certname in puppet land).

    ["and",
	["=","certname","<hostname>"],
	["=","type","Nagios::Nrpe::Service"]]


Once we have a list of nagios checks from your puppetdb, we just loop over each check.

    for c in r:
	    if only_check_name and only_check_name not in c['title']:
		    continue
        checks[c['title']] = c['parameters']['check_command']

Then we execute each check, letting the user know the output.

You can find the code at https://github.com/daniellawrence/nagioscheck
