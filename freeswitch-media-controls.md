## Relevant links

  + https://lists.freeswitch.org/pipermail/freeswitch-users/2013-January/091183.html

    Shows how `uuid_fileman` is bound to DTMF digits

  + [a 2013 issue with `uuid_fileman` (filed by the person in the above link)](https://webcache.googleusercontent.com/search?q=cache:mGd2A13Sl6QJ:https://freeswitch.org/jira/si/jira.issueviews:issue-html/FS-5026/FS-5026.html+&cd=3&hl=en&ct=clnk&gl=us)

  + [question on mailing list regarding `uuid_fileman` and `uuid_displace`](http://freeswitch-users.2379917.n2.nabble.com/uuid-displace-uuid-fileman-td7588618.html)

  + [question on mailing list regard `uuid_break`](https://marc.info/?l=freeswitch-dev&m=134126347227889)

  + [[Freeswitch-users] uuid_fileman seek ms](https://lists.freeswitch.org/pipermail/freeswitch-users/2013-March/094172.html)

## [FreeSWITCH `mod_commands`](https://freeswitch.org/confluence/display/FREESWITCH/mod_commands)

  + uuid_fileman

  + > pause
    > 
    >   Pause  <uuid> playback  of recorded  media that  was
    >   started with uuid_broadcast.
    >   
    >   Usage
    >   pause <uuid> <on|off>
    >   
    >   Turning  pause "on"  activates  the pause  function,
    >   i.e.  it  pauses  the playback  of  recorded  media.
    >   Turning pause  "off" deactivates the  pause function
    >   and resumes  playback of recorded media  at the same
    >   point where it was paused.
    >   
    >   Note: always returns -ERR  no reply when successful;
    >   returns -ERR No such channel! when uuid is invalid.

  + > uuid_audio
    > 
    > Adjust  the  audio  levels  on  a  channel  or  mute
    > (read/write) via a media bug.
    > 
    > Usage
    > uuid_audio <uuid> [start [read|write] [[mute|level] <level>]|stop]
    > <level> is in the range from -4 to 4, 0 being the default value.
    > 
    > Level is required for both mute|level params:
    > 
    > freeswitch@internal> uuid_audio 0d7c3b93-a5ae-4964-9e4d-902bba50bd19 start write mute <level>
    > freeswitch@internal> uuid_audio 0d7c3b93-a5ae-4964-9e4d-902bba50bd19 start write level <level>
    > 
    > (This command behaves funky. Requires further testing to vet all arguments. - JB)
    > 
    > See Also
    > 
    > mod_dptools: set audio level

  + > uuid_break
    > 
    > Break  out of  media being  sent to  a channel.  For
    > example,  if an  audio  file is  being  played to  a
    > channel,  issuing  uuid_break will  discontinue  the
    > media and  the call  will move  on in  the dialplan,
    > script, or whatever is controlling the call.
    > 
    > Usage: uuid_break <uuid> [all]
    > 
    > If   the   all  flag   is   used   then  all   audio
    > files/prompts/etc. that  are queued up to  be played
    > to the channel will be  stopped and removed from the
    > queue,  otherwise only  the currently  playing media
    > will be stopped.

  + uuid_debug_media
  + uuid_displace (? I guess this is supposed to mean "replace")

## Snippet from Joshua Young

```lua
local uuid = session:get_uuid();
api = freeswitch.API()
function func1()
    api:executeString("uuid_fileman " .. uuid .. " seek:-2000")
    session:destroy();
end
function func2()
    api:executeString("uuid_fileman " .. uuid .. " seek:+2000")
    session:destroy();
end
if argv[1] == "myarg1" then
    func1()
elseif argv[1] == "myarg2" then
    func2()
end
if session:ready() then
   session:answer()
   session:execute("sleep","1000")
   session:execute("bind_digit_action","myrealm,*,exec:lua,test.lua myarg1");
   session:execute("bind_digit_action","myrealm,#,exec:lua,test.lua myarg2");
   session:execute("bind_digit_action","myrealm,0,api:uuid_break," .. uuid);
   --session:execute("playback","/usr/local/freeswitch/sounds/joshebosh/Answering_Machine.wav")
   session:execute("playback","tone_stream://%(100,0,350);loops=-1");
end
```
