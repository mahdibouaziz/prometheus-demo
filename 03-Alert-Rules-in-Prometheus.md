# Alert Rules in Prometheus

Now we have Dashboards in Grafana that show the anomalies and let us visualize the data that we are interested in.

But in reality, people won't wait in front of the screen for anomalies.

When something happens in the cluster you need to get notified by email, slack, ... then you will check the Grafana Dashboards and analyse and fix the issue.

In this tutorial we will learn how to `Configure our monitoring stack to notify us whenever something unexpected happens`.

Alerting with Prometheus is separated into 2 parts:

1.  Define what we want to be notified about (`Alert Rules`)
2.  Send notification (`Alertmanager`)

# Take a look at Existing Alert Rules (deployed by default)

Go to the Prometheus UI and click on `Alerts`, this will list all of the alerts already configured alerts grouped by the name of the alert.

![Alt text](./images/alerts.png?raw=true)

Green alert == this alert is inactive or condition not met

Red alert = Firing or Condition is met

## Configuration Syntax of an Alert Rule

An alert has a:

- `name`: descriptive name about the alert.
- `expr`: the logical **expression** (the condition) defined in a `PromQL` syntax.
- `for`: defines the duration that Prometheus need to wait before sending an alert (maybe the Problem solves itself). For example in the Picture, Prometheus will check that the alert continues to be active for 10 minutes before firing the alert.
- `labels.severity`: contains a value like `critical`, `warning`, that defines the priority of the problem.
- `annotations`: specifies a set of descriptive information (additional information) that is gonna be sent if the alert is triggered.

an alert rule example:
![Alt text](./images/alert-rule.png?raw=true)

# Create our own `Alert Rules`
