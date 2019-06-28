### 0. `mod_signalwire`

It only has its config files in memory and in db files at `eval $${db_dir}`, that's all I could find out according to the [`mod_signalwire` docs](https://freeswitch.org/confluence/display/FREESWITCH/mod_signalwire).

TODO: figure out how to read those db files. No hint what RDBMS has been used to create that, and couldn't open with `sqlite3` (although that doesn't mean anything). Go through the [`mod_signalwire` source](https://freeswitch.org/stash/projects/FS/repos/freeswitch/browse/src/mod/applications/mod_signalwire/mod_signalwire.c). Yay.

Also didn't find docs for the `signalwire` executable:
```text
freeswitch@tr2> signalwire

[             adopted]  [            adoption]  [               debug]  [               kslog]
[              reload]  [               token]  [              update]
```

### 1. Dynamic dialplan (or how to make changing the dialplan managable)

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

  ANSWER: See https://github.com/society-for-the-blind/TR2 as it does exactly that.

### 2. "Speech Phrase Management" in docs VS "Language Management" in demo config

I think "Language Management" has evolved into "Speech Phrase Management", but the source got left behind. Basing this on

 + Both manage phrases, but the latter properly describes this fact

 + The `<phrases>` tag is missing form the latter, because the main configuration section is now called "phrases"

> **QUESTION**: The syntax of the `<language>` tag differs in both versions (see below). Which one is correct? Where is the documentation for that?
>
> The [Phrases Section Primer](https://freeswitch.org/confluence/display/FREESWITCH/Speech+Phrase+Management#SpeechPhraseManagement-PhrasesSectionPrimer) shows one answer, but because our current FreeSWITCH instance runs with the demo config and still works, what is the **definitive syntax**?

> **QUESTION**: **How to change language?** For example, the IVR has a menu offering different other languages.
> Try out `<action application="set" data="default_language=en"/>` described in [Selecting the language](https://freeswitch.org/confluence/display/FREESWITCH/Speech+Phrase+Management#SpeechPhraseManagement-Selectingthelanguage).
> Does it work with `play_and_get_digits` for example?
> Does it operate on the `name` attribute of the [`<language>` tag](https://freeswitch.org/confluence/display/FREESWITCH/Speech+Phrase+Management#SpeechPhraseManagement-PhrasesSectionPrimer)?

> **QUESTION**: How to change language **in Lua script**?
>
> Everything in the previous questions seems to be relevant, because one can call [`playAndGetDigits`](https://freeswitch.org/confluence/display/FREESWITCH/Lua+API+Reference#LuaAPIReference-session:playAndGetDigits) or [`freeswitch.IVRMenu`](https://freeswitch.org/confluence/display/FREESWITCH/Lua+API+Reference#LuaAPIReference-freeswitch.IVRMenu) from a Lua script, and both can be configured phrases, just as their XML counterparts.
>
> But why is there a [`session:sayPhrase`](https://freeswitch.org/confluence/display/FREESWITCH/Lua+API+Reference#LuaAPIReference-session:sayPhrase) function then, that has an explicit `language` argument, and its doc section describes how to reimplement `playAndGetDigits`?
>
>>  `session:sayPhrase(macro_name [,macro_data] [,language]);`
>>
>>  + macro_name - (string) The name of the say macro to speak.
>>  + macro_data - (string) Optional. Data to pass to the say macro.
>>  + language - (string) Optional. Language to speak macro in (ie. "en" or "fr"). Defaults to "en".
>>
>>  To capture events or DTMF, use it in combination with [`session:setInputCallback`](https://freeswitch.org/confluence/display/FREESWITCH/Lua+API+Reference#LuaAPIReference-session:setInputCallback).
>>
>>  When used with setInputCallback, the return values and meanings are as follows:
>>
>>  + true or "true" - Causes prompts to continue speaking.
>>  + Any other string value interrupts the prompt.

From the demo config in `/etc/freeswitch/freeswitch.xml` (after expanding the "include" section):

```xml
<document type="freeswitch/xml">¬
  <!-- languages section (under development still) -->
  <section name="languages" description="Language Management">
    <language name="en" say-module="en" sound-prefix="$${sound_prefix}" tts-engine="cepstral" tts-voice="callie">
      <phrases>
        <macros>
          <macro name="demo_ivr_count">
            <input pattern="^(\d+)$">
              <match>
                <action function="play-file" data="voicemail/vm-you_have.wav"/>
                <action function="say" data="$1" method="pronounced" type="name_spelled"/>
                <action function="play-file" data="voicemail/vm-messages.wav"/>
              </match>
            </input>
          </macro>
          <!-- ... -->
        </macros>
        <macros name="voicemail_ivr">
          <macro name="enter_id">
            <!-- ... -->
          </macro>
          <macro name="enter_pass">
            <!-- ... -->
          </macro>
          <!-- ... -->
        </macros>
      </phrases>
    </language>
  </section>
```

"Speech Phrase Management" from the [docs](https://freeswitch.org/confluence/display/FREESWITCH/mod_dptools%3A+phrase#mod_dptools:phrase-Sample):

```xml
<section name="phrases" description="Speech Phrase Management">
   <macros>
     <language name="en" sound_path="/snds" tts_engine="cepstral" tts_voice="david">
       <macro name="msgcount">
         <input pattern="(.*)">
           <action function="execute" data="sleep(1000)"/>
           <action function="play-file" data="vm-youhave.wav"/>
           <action function="say" data="$1" method="pronounced" type="items"/>
           <action function="play-file" data="vm-messages.wav"/>
         </input>
       </macro>
       <macro name="saydate">
         <input pattern="(.*)">
           <action function="say" data="$1" method="pronounced" type="current_date_time"/>
         </input>
       </macro>
       <macro name="timespec">
         <input pattern="(.*)">
           <action function="say" data="$1" method="pronounced" type="time_measurement"/>
         </input>
       </macro>
       <macro name="spell">
         <input pattern="(.*)">
           <action function="say" data="$1" method="pronounced" type="name_spelled"/>
         </input>
       </macro>
       <macro name="spell-phonetic">
         <input pattern="(.*)">
           <action function="say" data="$1" method="pronounced" type="name_phonetic"/>
         </input>
       </macro>
       <macro name="tts-timeleft">
         <input pattern="(\d+):(\d+)">
           <action function="speak-text" data="You have $1 minutes, $2 seconds remaining $strftime(%Y-%m-%d)"/>
         </input>
       </macro>
     </language>
     <language name="fr" sound_path="/var/sounds/lang/fr/jean" tts_engine="cepstral" tts_voice="jean-pierre">
       <macro name="msgcount">
         <input pattern="(.*)">
           <action function="play-file" data="tuas.wav"/>
           <action function="say" data="$1" method="pronounced" type="items"/>
           <action function="play-file" data="messages.wav"/>
         </input>
       </macro>
       <macro name="timeleft">
         <input pattern="(\d+):(\d+)">
           <action function="speak-text" data="il y a $1 minutes et de $2 secondes de restant"/>
         </input>
       </macro>
     </language>
   </macros>
 </section>
```

#### 2.0 Relevant articles in the docs (what I could find, at least)

##### 2.0.0 [Understanding the Configuration Files](https://freeswitch.org/confluence/display/FREESWITCH/Understanding+the+Configuration+Files) (updated: 2018.10.01)

Shows "Language Management" in [XML](https://freeswitch.org/confluence/display/FREESWITCH/Understanding+the+Configuration+Files#UnderstandingtheConfigurationFiles-XML) section.

There is a [languages](https://freeswitch.org/confluence/display/FREESWITCH/Understanding+the+Configuration+Files#UnderstandingtheConfigurationFiles-languages) section, but it is empty.

##### 2.0.1 [mod_dptools: phrase](https://freeswitch.org/confluence/display/FREESWITCH/mod_dptools%3A+phrase) (updated: 2016.11.12)

Shows the "Speech Phrase Management" XML example pasted above, and contains a link to [Speech Phrase Management](https://freeswitch.org/confluence/display/FREESWITCH/Speech+Phrase+Management).

##### 2.0.2 [Speech Phrase Management](https://freeswitch.org/confluence/display/FREESWITCH/Speech+Phrase+Management) (updated: 2016.11.12)

> **QUESTION**: This seems to be the de facto documentation on how to use phrases, but why isn't this included in the current demo config since 2016?
>
> TODO: check FreeSWITCH repo for the dates when the demo configs have last been updated.

##### 2.0.3 [Configuring FreeSWITCH](https://freeswitch.org/confluence/display/FREESWITCH/Configuring+FreeSWITCH) (updated: 2018.12.30)

Does not mention "Language Management" at all. Relevant parts:

 + search for "language", 1 result:

   [Configuration Files](https://freeswitch.org/confluence/display/FREESWITCH/Configuring+FreeSWITCH#ConfiguringFreeSWITCH-ConfigurationFiles) mentions the `lang` directory

 + search for "phrase", 3 results:

   [freeswitch.xml](https://freeswitch.org/confluence/display/FREESWITCH/Configuring+FreeSWITCH#ConfiguringFreeSWITCH-freeswitch.xml) section mentions the "phrases section" and the [Example freeswitch.xml](https://freeswitch.org/confluence/display/FREESWITCH/Configuring+FreeSWITCH#ConfiguringFreeSWITCH-Examplefreeswitch.xml) section shows the "Speech Phrase Management" XML snippet.

##### 2.0.3 [Default Configuration](https://freeswitch.org/confluence/display/FREESWITCH/Default+Configuration) (updated: 2018.12.30)

The main diagram shows "Speech Management", and the only other relevant section is [conf/lang/](https://freeswitch.org/confluence/display/FREESWITCH/Default+Configuration#DefaultConfiguration-conf/lang/), which is very short with very little information.
