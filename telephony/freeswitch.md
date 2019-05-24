### `mod_signalwire`

It only has its config files in memory and in db files at `eval $${db_dir}`, that's all I could find out according to the [`mod_signalwire` docs](https://freeswitch.org/confluence/display/FREESWITCH/mod_signalwire).

TODO: figure out how to read those db files. No hint what RDBMS has been used to create that, and couldn't open with `sqlite3` (although that doesn't mean anything). Go through the [`mod_signalwire` source](https://freeswitch.org/stash/projects/FS/repos/freeswitch/browse/src/mod/applications/mod_signalwire/mod_signalwire.c). Yay.

Also didn't find docs for the `signalwire` executable:
```text
freeswitch@tr2> signalwire

[             adopted]  [            adoption]  [               debug]  [               kslog]
[              reload]  [               token]  [              update]
```

