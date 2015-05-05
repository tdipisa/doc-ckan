.. _troubleshooting:

###############
Troubleshooting
###############

====
CKAN
====

Resource visualization returns a server error
---------------------------------------------

If the log file reports a::

   (ProgrammingError) permission denied for relation _table_metadata
   
try resetting the ``select`` grants::

   su - postgres -c "psql datastore"
   GRANT SELECT ON ALL TABLES IN SCHEMA public TO datastore;
   GRANT SELECT ON ALL TABLES IN SCHEMA public TO datastorero;


Harvest: error in fetching
--------------------------

The fetch log (at ``/var/log/ckan/fetch.log``) may present errors like this one::

   Traceback (most recent call last):
     File "/usr/lib/ckan/default/bin/paster", line 9, in <module>
       load_entry_point('PasteScript==1.7.5', 'console_scripts', 'paster')()
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/script/command.py", line 104, in run
       invoke(command, command_name, options, args[1:])
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/script/command.py", line 143, in invoke
       exit_code = runner.run(args)
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/script/command.py", line 238, in run
       result = self.command()
     File "/usr/lib/ckan/default/src/ckanext-harvest/ckanext/harvest/commands/harvester.py", line 126, in command
       for method, header, body in consumer.consume(queue='ckan.harvest.fetch'):
     File "/usr/lib/ckan/default/src/ckanext-harvest/ckanext/harvest/queue.py", line 160, in consume
       self.redis.set(self.persistance_key(body),
     File "/usr/lib/ckan/default/src/ckanext-harvest/ckanext/harvest/queue.py", line 165, in persistance_key
       return self.routing_key + ':' + message[self.routing_key]
   TypeError: cannot concatenate 'str' and 'NoneType' objects

It should be caused by changes in an harvesting source.

- Go in the CKAN admin page for the given source.
- Remove current jobs ( admin > clear > confirm )
- From the command line, run the command::
   redis-cli flushall
- As root user, restart CKAN ::   
   service supervisord restart

    
