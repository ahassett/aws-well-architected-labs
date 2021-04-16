---
title: "Simulate Application Issue"
date: 2020-04-24T11:16:09-04:00
chapter: false
weight: 2
pre: "<b>2. </b>"
---

Being able to understand the health of your workload is essential in building a reliable operational practice for your workload. To do this, you need to adopt the appropriate monitoring tool to gain visibility and analyze your workload state, along with defining the right metric based on identified key performance indicator. This will create an opportunity you to streamline it with automatic detection and alerting that will allow you to detect issue or potential issue early and react promptly to remediate.

In the previous section, we have deployed sample application API running within in a VPC. Now, in this section we will simulate a performance issue and trigger an alarm notifying our the operations team. In our scenario thr SLA for our API response is defined to be under 6 seconds, if out API response takes longer than 6 seconds broader user experience is severely impacted, hence our operations team needs to be alerted to response to this. 

To do this we want to continuously monitor our API, and emulate what our customer experience in our API.
We can do this by deploying a Canary monitor using [CloudWatch Synthetics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html) to continuously check the API response time. If the duration of the check is taking longer than our defined SLA, we will trigger a CloudWatch alarm that will in turn send notification to our Operation team..  

The resources needed for this section is already deployed along with the cloudformation template of your application in previous section. But please follow along below instructions to simulate the API issue and gain understanding on what happen 

To proceed with this section, please ensure you have completed Section 2 of this lab.

![Section3 Base Architecture](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-testing-canary-alarm-architecture.png)

### 3.1 Sending traffic to Application

In this part of the lab we will be sending multiple concurrent requests to the application to simulate large incoming traffic. This is intended to overwhelm the service, and as time goes, response time to the application will become slower thus, setting of the CloudWatch Alarm that will notify our Operations team via email.

From the cloud9 terminal do the following :

```
cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/scripts/
```

Confirm that you have the `test.json` in the above folder that contains this text.

```
{"Name":"Test User","Text":"This Message is a Test!"}
```



Once that's confirmed go to cloudformation console, and take note of the **OutputApplicationEndpoint** value under Output tab of `walab-ops-sample-application` stack. Then paste the value into the command below.

![Section3 Succces Screenshot](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-stackoutput.png)

```
bash simulate_request.sh '<OutputApplicationEndpoint value you copied before>'
```

This script uses Apache Benchmark to send 60000000 requests at 3000 concurrently.

Once you executed this command, you will see the command output gradually from successful 200 response 

![Section3 Succces Screenshot](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-success-traffic-requests.png)

To several 504 time out response, this is intended, as we are essentially flooding the API.
The requests we are generating is overwhelming our application triggering the occasional Timeout generated by our Load Balancer.

![Section3 Failure Screenshot](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-failure-traffic-requests.png)

Keep the command running in the background throughout the lab and proceed to the next step.

### 3.2 Observing the alarm being triggered.

After approximately 6 minutes you should now see an alarm being triggered, and if you've confirmed subscription on the SNS topic, you should be seeing an email being generated as below, indicating the CloudWatch alarm that has been triggered.

![Section3 Email](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-email.png)
 
You can check alarm this by going to the Cloudwatch console, and click on the Alarms section on the left menu.
Click on the Alarms called `mysecretword-canary-duation-alarm` that should currently be in an Alarm State.

![Section3 Failure Screenshot](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-alarm.png)

Clicking on the Alarm will display the CloudWatch metrics that the alarm is triggering from.
Which is the `Duration` metric emitted by `CloudWatchSynthetics` with CanaryName `mysecretword-canary`
This metric essentially measures how it takes for the canary requests until a response is received by the application. The alarm is triggered whenever the value of the `Duration` metric is above 6 seconds within 1 minute duration.

![Section3 Failure Screenshot](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-alarm-detail.png)

### 3.3 Observing the Canary.

Working backwards from the alarm, let's now take a look at the Canary monitoring the API emitting those metrics.

Monitoring application health, can be done from multiple angles, system / applications logs, metrics operating system, and services the application is running on. None of these however will be able to replicate accurately the a point of view from the consumer of your application. 

This is why in this scenario a canary is implemented to regularly call and check the health of the API from it's external endpoint using [ CloudWatch synthetics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html)



To look at the Canary configuration, go to CloudWatch Console, and click on **Synthetics** and **Canaries** on the left menu. Locate and click the Canary called `mysecret-canary` 

![Section3 Canary](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-canary.png)

Under **Configuration** tab you can see the Canary configuration a snippet of the Canary Script.
scrolling down to the `requestOptionStep1` variable of the script, you can see that the script is calling the `/encrypt` action of the API via the load balancer endpoint, passing a dummy payload.   

![Section3 Canary](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-canary-detail.png)

And under **Monitoring** tab you will be able to see a visualization of the Duration metric that is being used to trigger our alarm.

![Section3 Canary](/Operations/200_Automating_operations_with_playbooks_and_runbooks/Images/section3-canary-monitor.png)


This concludes **Section 2** of this lab. 
With this section concluded we have simulated a large incoming traffic to our application API overwhelming the application, causing the API to response slowly and occasionally timed out. In a real operational scenario, this scenario would trigger an incident notification alerting the operations team to take action.

So now,let's proceed to the next Section, where we will be building the Automated playbook to investigate issue in the application. 

{{< prev_next_button link_prev_url="../2_configure_ecs_repository_and_deploy_application_stack/" link_next_url="../3_build_execute_investigative_playbook/" />}}

___
**END OF SECTION 2**
___