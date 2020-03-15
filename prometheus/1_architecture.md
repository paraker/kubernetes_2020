# client libraries
`Client libraries` are used to put together metrics from apps you own.<br>
Prometheus can scrape then scrape these metrics.<br>
The libraries are unique per coding language used in your app.<br>
Collecting this data is called `instrumenting`.<br>

You add `instrumentation` to your code, and to do this you need to have the `libraries`.
The output should/does most often end up in a `/metrics endpoint` that `Prometheus` can scrape through HTTP.

Prometheus provides the following libraries officially:
* Go
* Java & Scala
* Python
* Ruby

Third party libraries exist for almost any other language, such as `bash`, `c++`, `node.js` etc etc.

# exporters
`exporters` are used to instrument applications that we don't have the source code for.<br>
`Instrumentation` for `exporters` are known as `custom collectors` or `ConstMetrics`.<br>
<br>
exporters come in different categories:
* DB
* Hardware/Software (OS)
* Messaging (HTTP etc)
* Monitoring systems (JMX/Cloudwatch etc)

Examples of systems that exporters can be used on are `MySQL`, `Host/OS`, `HAProxy`.<br>

`Prometheus` sends a request to a `exporter target`.<br>
The `exporter` then goes and fetches data, formats the data  and feeds data back to `Prometheus`.


