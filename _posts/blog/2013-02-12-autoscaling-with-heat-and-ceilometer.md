---
layout: post
title: Autoscaling with Heat and Ceilometer
category: blog
description: Like AWS CloudFormation, Heat allows to create auto scaling stacks.
---

The post from  http://techs.enovance.com/5991/autoscaling-with-heat-and-ceilometer

Like AWS CloudFormation, Heat allows to create auto scaling stacks. In order to do this, some metrics need to be retrieved from your VM and some actions need to be triggered when a specified event occurs on these metrics. These actions are usually upscaling (create some new VMs) or downscaling (destroy some VMs).

In OpenStack Grizzly, a simplistic system was put in place that could mostly serve demonstration needs but could not be used for most real world scenarios: the metrics were retrieved via an agent running inside each VM associated to the Auto Scaling group, and were stored in the Heat database. And the alarms that are triggered up/down auto scaling were stored in the Heat database too.

Thinking about it, it seemed clear that Heat should be a tool which only does orchestration on different API, not storing/handling alarms and metrics, under the UNIX old principle: do one thing and do it well.

It also occurred that Ceilometer already records metrics from Openstack components for billing purpose and, after discussion between Heat and Ceilometer project teams around the Havana Summit, it was decided that it would make more sense for Heat to rely on Ceilometer Alarming. This led us to validate a series of blueprints which are now mostly implemented. Starting with the Havana version (currently trunk), Ceilometer can now trigger actions when something happens to these metrics by creating alarms. The Heat agent inside the VM is no more needed.

##Ceilometer

During the Grizzly cycle, I happened to write an Alarming function prototype which was used to define the plan for Havana. The alarming plan we agreed on for the Ceilometer part is here, thanks to the great design proposal that Eoghan Glynn presented at the Summit and primed the discussions pretty well.

All pieces of the puzzles to have a working alarming have been implemented for havana-2, but the solution is not yet fully scalable. Mainly written by Eoghan Glynn (RedHat), Angus Salkeld (RedHat), Julien Danjou (eNovance) and Mehdi Abaakouk(me, eNovance). The work done by eNovance was mostly sponsored by CloudWatt, as they saw early on the need to offer auto scaling capabilities to their customers.

The alarming plan is broken down into many components:

* It shares the common Ceilometer database API to store the alarms definition.
* The Ceilometer-api v2 has been extended to allow to get/create/modify/delete alarms.
* A service ‘ceilometer-alarm-singleton’ does the threshold evaluation of each alarms. If the threshold is reached, this service uses the alarm notifier RPC API to trigger the actions associated to the alarms.
* A service ‘ceilometer-alarm-notifier’ listens to the RPC bus for alarm notification events and executes the triggered actions. The implemented action schemes are currently log:// to write alarm state changes to file, http(s):// to call a POST request to an external API (this is the one used by Heat). More schemes will be added in the future. In the case of the http method, this included some simple https pki certificate validation to avoid risks of spoofing.

Targeted for Havana, the last parts to implement are:

* Make it scalable by distribute the alarm threshold evaluation across many servers
* Allow to create alarms that are combination of other alarms state.
* Having a Cloudwatch Compatible API

##Heat

On the Heat side, the main work was to write the new resource type OS::Metering::Alarm for Ceilometer alarming. And map the AutoScalingGroup tags to the nova user metadata of each VM to allow the creation of alarms which match the whole AutoScaling group. This work has been done by Angus Salkeld (Redhat). This resource allows to create Ceilometer alarms via a Heat template. A sample template can be found here.

Some remaining works on havana-3 just landed at review.openstack.org to allow the use of Ceilometer for the resource AWS::CloudWatch::Alarm instead of the old light Cloudwatch implementation of Heat. With this change, the cloud provider can choose between using Ceilometer for Cloudwatch/CloudFormation or keeping the previous light implementation. The remaining point to solve for Heat, is to allow a non-admin user to generate the EC2 credentials for the ‘CfnUser’ (user used to spawn/delete VMs)

##Give it a try !

Boot a vm, install devstack, configure your stack. Enable all Heat/Ceilometer services in your localrc:


* enable_service ceilometer-acentral,ceilometer-collector,ceilometer-api,ceilometer-acompute,ceilometer-alarm-singleton,ceilometer-alarm-notifier
* enable_service heat,h-api,h-api-cfn,h-api-cw,h-eng
* EXTRA_OPTS=(notification_driver=nova.openstack.common.notifier.rabbit_notifier,Ceilometer.compute.nova_notifier)
And go !

    $ ./stack.sh

A couple of minutes later we can start playing. You can take a look at this quick video demo of the feature

##How both work together

When Heat creates the auto scaling system with the basic template:

* A Load Balancer is created (a VM with haproxy inside, not yet the LBAAS of quantum^Wneutron)
* A first VM of the Auto Scaling Group (AG) with AG tags injected as nova user metadata, to easily identify the group in Ceilometer
* Two alarms (one for downscaling, one for upscaling), each alarm is configured to match cpu_util metrics of the AG and to call back the Heat API to trigger the upscaling or the downscaling.

Next, in Ceilometer, the metering records the OpenStack metrics and the alarming evaluates each alarm to see if the threshold is reached. If it’s reached, Ceilometer calls the alarm action. For the autoscaling system, this action (previously configured by Heat) calls back the Heat API to trigger the Up (or Down) scaling. And when Heat receives this API call, it starts (or destroys) a VM.

##Some notes about deploying alarming

When Ceilometer is used for alarming and billing, it will be preferable to have two distinct mongodb databases, one for billing and one for alarming, in order to configure different db/pipeline parameters:

* Billing needs to keep a particular history (long db-ttl), because it needs to be sure that samples arrived
* Alarms need to have a small history of samples (short db-ttl, 15 days?), because it we will seldom have to fetch historical data over a lager period of time for alarming purposes We can also note that, in the case of Alarming, and unlike billing, if some samples are missing it is not as important for the overall process.

Another good reason to split the database is that you may not want an end-user’s API with the ability to write to a billing database, even though we have done our best efforts to ensure proper isolation of user provided meters.

An additional useful “trick” is to customize the Ceilometer pipeline in order to have some metrics for billing only, and others for both, which can be done since the Grizzly version. But to split the databases, we’ĺl need to wait a bit for the completion of theblueprint alarming-metering-separation.
