Auditbeat Demo for CVE-2017-5638
---

This demonstrates how the [file_integrity][file_integrity] module in Elastic's
Auditbeat can be used to find machines that have the Apache Struts jar.

Then we exploit the vulnerability in Apache Struts and detect the executions
using Auditbeat's [auditd][auditd] module.

[file_integrity]: https://www.elastic.co/guide/en/beats/auditbeat/current/auditbeat-module-file_integrity.html
[auditd]: https://www.elastic.co/guide/en/beats/auditbeat/current/auditbeat-module-auditd.html

### Usage

Start Elasticsearch, Kibana, and install the Auditbeat dashboards.

`docker-compose up`

Start and provision a Debian 9.

`vagrant up`

The Vagrant machine will have:

- Auditbeat
- Tomcat 7
- Apache Struts Showcase Webapp

Run the exploit.

`python exploit.py '/usr/bin/touch your-box-has-been-pwned'`

Open Kibana on the host machine.

http://localhost:5601

### View Results

[prevent]: http://localhost:5601/app/kibana#/discover/a380a060-cb44-11e7-9835-2f31fe08873b?_g=(refreshInterval:(display:Off,pause:!f,value:0),time:(from:now-1y,mode:quick,to:now))&_a=(columns:!(file.path,event.action),filters:!(('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'auditbeat-*',key:event.module,negate:!f,params:(query:file_integrity,type:phrase),type:phrase,value:file_integrity),query:(match:(event.module:(query:file_integrity,type:phrase))))),index:'auditbeat-*',interval:auto,query:(language:lucene,query:'file.path.raw:%2F.*struts.*%5C.jar%2F'),sort:!('@timestamp',desc))
[detect]: http://localhost:5601/app/kibana#/discover/d382f5b0-c1c6-11e7-8995-936807a28b16?_g=(refreshInterval:(display:Off,pause:!f,value:0),time:(from:now-1y,mode:quick,to:now))&_a=(columns:!(beat.hostname,process.args,auditd.summary.actor.primary,auditd.summary.actor.secondary,process.exe),filters:!(('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'auditbeat-*',key:event.module,negate:!f,params:(query:auditd,type:phrase),type:phrase,value:auditd),query:(match:(event.module:(query:auditd,type:phrase)))),('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'auditbeat-*',key:event.action,negate:!f,params:(query:executed,type:phrase),type:phrase,value:executed),query:(match:(event.action:(query:executed,type:phrase)))),('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'auditbeat-*',key:auditd.summary.actor.secondary,negate:!f,params:(query:tomcat,type:phrase),type:phrase,value:tomcat),query:(match:(auditd.summary.actor.secondary:(query:tomcat,type:phrase))))),index:'auditbeat-*',interval:auto,query:(language:lucene,query:'*'),sort:!('@timestamp',desc))

[Find all Struts Jars][prevent]

![Auditbeat File Integrity Search](images/auditbeat-find-struts.png)

[See execve syscalls by the tomcat user][detect]

![Auditbeat Execve Search](images/auditbeat-detect-tomcat-execve.png)
