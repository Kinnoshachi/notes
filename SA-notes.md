# SNS

## ACG notes 
https://acloud.guru/course/aws-certified-solutions-architect-associate/learn/applications/sns/watch

#### Can push notifications as SMS to cell, email, SQS, http endpoints 
#### Topic is an access point that users can subscribe to. A topic can deliver to multple endpoint types. All SNS stored reduntantly accorss AZs.
#### instataneupus push based delivery
#### SNS is push based, SNS is pulled based using ec2s.

## SA Study Guide notes
#### SNS based on publish-subscribe (pub-sub) messaging paradigm
#### Apple, Android, Fire OS, and Windows devices, In China, to Android devices with Baidu Cloud Push. (SMS) messages to mobile device users in the United States or to email recipients worldwide

#### two types of clients: publishers and subscribers (sometimes known as producers and consumers)

![image1](https://learning.oreilly.com/library/view/aws-certified-solutions/9781119138556/images/ec08f004.jpg)

* relay events in workflow systems among distributed computer applications, 
* move data between data stores, or update records in business systems
* Event updates and notifications concerning validation, approval, inventory changes, and shipment status are immediately delivered to relevant system components and end users
* relay time-critical events to mobile applications and devices

* Application and System Alerts
* Push Email and Text Messaging
* Mobile Push Notifications, directly to mobile applications

### Fanout
#### parallel asynchronous processing by sending to multiple places. The illustration; order placed, one ec2 endpoint starts processing the order, 2nd endpoint ec2 send order data to DW app for analysis, or data can be replicted to dev env for further optimization.
![sns image](https://learning.oreilly.com/library/view/aws-certified-solutions/9781119138556/images/ec08f005.jpg)


["SNS"](https://github.com/Kinnoshachi/notes/blob/master/SA-notes.md#sns)
