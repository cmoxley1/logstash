# Skills test solution write up 

## Resources used for learning:
I used many of these resources to get to the end product.

* Youtube videos (too many to list)  
* www.elastic.co/guide/en/logstash/7.15/index.html 
* https://grokdebug.herokuapp.com/
* https://medium.com/statuscodeusing-custom-regex-patterns-in-logstash-fa3c5b40daab
* https://logstashbook.com/
* https://logz.io/blog/logstash-grok/
* https://datatracker.ietf.org/doc/rfc5424/
* https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns
*  **GOOGLE**
   

## Steps taken to get to the final 
#### 1. Creating the config file
I started looking at many of the above resources to get going. I first installed logstash on my MAC using brew. I got the understand that I needed to create a config file with input, filter and output as the foundation of my config file. 

```CONF
input{}
filter {}}
output {}
```
#### 2. Ingesting the log data
Based on the requirements of the skills test.  I needed to input provided security log data via and Logstash input plugin. I saw a lot of examples using the file plugin. This is the direction I went.
```CONF
input {
    file {
        path => "./logsample.log" 
    }
}
```
#### 3. Output the file data
Again I used the file plugin to output the data to a json file. 
```CONF
output{
    file {
        path =>./outfile.json" 
    }
}
```
#### 4. Filter making steps. Where the first mistakes where made. 
This is where I started my filter plugin 
```CONF
filter {
    grok{

    }
}
```
I used  https://grokdebug.herokuapp.com/ to help me debug my GROK filter. This is where I made mistakes that took me awhile to figure out. At the time I did not know it was typos in my code plus the GROK filter that was wrong. Just because the GROK debugger will let you do something does not mean logstash will let you while actual running the code. This made me think it was a issue I was having on my MAC, so I swithced to my Windows machine which introduced issues like using / in the path variable and not \ . 

Here is my first attempt at my GROK filter 
```
{TIMESTAMP_ISO8601:timestamp} %{DATA:UNWANTED} %{DATA:UNWANTED} \- \- alertname=\"%{QS:description}\" 
computername=\"%{QS:hostname}\"  computerip=\"%{IP:source_ip}\" severity=%{QS:severity}" }
 #mutate {
```
I later learned that there are better options to use and the GROK does not like escapes. I beleive I saw them in dissect examples. (I could be wrong.) 

Also using quotedstring introduced the problem of removing quotes I did not want. 
```
%{QS:test}
```
This one seemed to do the trick with seleeting alertname. 
```
%{DATA:test}
```

Another issue I had was my output had a GROKFAILURE output with the json. I figured out that I had a few empty lines in the logsample.log file causing thie repsponse.

## Final results
### MAC
```CONF
input {
  file {
    path => "/Users/outside/Documents/GitHub/logstash_new/logsample.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
  grok {
    match => { "message" => ['%{SYSLOG5424BASE} alertname="%{DATA:description}" computername="%{HOSTNAME:hostname}" computerip="%{IP:source_ip}" severity="%{INT:severity}'] }
  }
  translate {
    source => "severity"
    target => "severity"
    dictionary => {
      "1" => "High"
    }
  }
}  

output {
  file {
    path => "/Users/outside/Documents/GitHub/logstash_new/outputfile_mac.json"
    codec => "json"
    write_behavior => "overwrite"
  }
  stdout { }
  }
```
### Windows

```CONF
input {
    file {
        path => ["C:/Users/chase/OneDrive/Documents/GitHub/logstash/logsample.log"]
        start_position => "beginning"
        sincedb_path => "nul"
    }
}
filter {
  grok {
    match => { "message" => ['%{SYSLOG5424BASE} alertname="%{DATA:description}" computername="%{HOSTNAME:hostname}" computerip="%{IP:source_ip}" severity="%{INT:severity}'] }
  }
  translate {
    source => "severity"
    target => "severity"
    dictionary => {
      "1" => "High"
    }
  }
}  
output {
    file { 
        path => ["C:\Users\chase\OneDrive\Documents\GitHub\logstash\outfile_win.json"]
        codec => "json"
        write_behavior => "overwrite"
    }
    stdout { }
}
```
## OUTPUT
```JSON
{
         "@timestamp" => 2021-11-01T20:38:46.205Z,
    "syslog5424_proc" => "2496",
          "source_ip" => "216.58.194.142",
            "message" => "<14>1 2016-12-25T09:03:52.754646-06:00 contosohost1 antivirus 2496 - - alertname=\"Virus Found\" computername=\"contosopc42\" computerip=\"216.58.194.142\" severity=\"1\"",
           "@version" => "1",
        "description" => "Virus Found",
           "hostname" => "contosopc42",
           "severity" => "High",
     "syslog5424_app" => "antivirus",
     "syslog5424_ver" => "1",
               "host" => "CMsMacBookPro.attlocal.net",
    "syslog5424_host" => "contosohost1",
               "path" => "/Users/outside/Documents/GitHub/logstash_new/logsample.log",
     "syslog5424_pri" => "14",
      "syslog5424_ts" => "2016-12-25T09:03:52.754646-06:00"
}
```
!(output.jpg)