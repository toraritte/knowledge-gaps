### `mod_signalwire`

It only has its config files in memory and in db files at `eval $${db_dir}`, that's all I could find out according to the [`mod_signalwire` docs](https://freeswitch.org/confluence/display/FREESWITCH/mod_signalwire).

TODO: figure out how to read those db files. No hint what RDBMS has been used to create that, and couldn't open with `sqlite3` (although that doesn't mean anything). Go through the [`mod_signalwire` source](https://freeswitch.org/stash/projects/FS/repos/freeswitch/browse/src/mod/applications/mod_signalwire/mod_signalwire.c). Yay.

Also didn't find docs for the `signalwire` executable:
```text
freeswitch@tr2> signalwire

[             adopted]  [            adoption]  [               debug]  [               kslog]
[              reload]  [               token]  [              update]```

### Dynamic dialplan (or how to make changing the dialplan managable)

+ https://lists.freeswitch.org/pipermail/freeswitch-users/2012-April/082857.html
  > I'd also "cut" the dialplan between simple parts (on disk) and subject to changes parts (on DB).

  How?

+ https://lists.freeswitch.org/pipermail/freeswitch-users/2012-April/082854.html
  > 1) using lua or(python?) to serve dialplan instead of mod_xml_curl
  > 2) writing a dialplan module(something like mod_xml_curl) in c which will
  > do all
  > the db lookups etc..
  > 3) using mod_erlang_event in outbound mode & spawning several erlang
  > workers to do db lookups etc..
  > 4) using mod_event_socket in outbound mode & making db lookups on a
  > different server.




