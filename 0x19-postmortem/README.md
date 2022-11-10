## My first postmortem

# Issue Summary

From 5:24 PM to 7:10 PM UTC +1, requests to my API(proect_mg api) resulted in 500 error response messages. The few web applications that rely on this API also returned errors and functionality was affected. Users were delayed from meeting up with their deadlines of their collaboration tasks . At its height, the issue affected 100% of traffic to this API infrastructure. The root cause of this incidence was a wrong logical configuration change that caused a bug.

# Timeline (UTC +1)
4:30 PM: Configuration push begins.
5:24 PM: Issue detected by senior software engineer of “Alpha-Red” team.
5:46 PM: Users sends compaints of a 500 error response.
5:58 PM: The last push, a configuration change was investigated. 
6:04 PM: Incident escalated to “Biza” team and failed configuration change was rolled back. 
6:16 PM: Debugging begins.
6:35 PM: Successful configuration change rolled back.
6:49 PM: Server restarts begin.
6:55PM: Users sends feedbacks on api functioning well.
7:10 PM: 100% of traffic back online.

# Root cause 

At 4:30 PMUTC +1, a configuration change was unintentionally released to the production environment without any form of testing. The change specified which users has certain access to manipulate some data in the base with respect to their roles. This was done by writing a configuration file where the process of verifying roles was written to be an asynchronous function by uing the “async method” without using the operator “await”  . The call to the async method starts an asynchronous task. However, because no “await” operator is applied, the program continues without waiting for the verification task to be completed. This behavior isn't expected as in most cases. This exposed a bug in the authorization process which caused all users to have no access to manipulating the data stored in the database and hence unable to carry out the tasks assigned to them within the particular time frame allocated. The combination of the bug and configuration error quickly caused all of the serving threads to be consumed. Traffic was permanently queued waiting for a serving thread to become available. Concurrency was not attained and this led to the incidence of a “500 error response message”.

# Resolution and recovery

At 5:24 PM PT, the monitoring systems alerted the lead software engineer of the”Alpha-Red” team who investigated and quickly escalated the incident to the “Biza team. By 6:09PM,  Biza team identified that the monitoring system was exacerbating the problem caused by this bug.

At 6:11 PM, an attempt to rollback the problematic configuration change was made. This rollback failed due to complexity in the configuration system which caused the security checks to reject the rollback. These problems were addressed and we successfully rolled back at 6:15 PM.

At 6:16PM, debugging started and the await operator was added to make the async function run properly. The verification function got fixed and this time, a test was written to check the function is working exactly the way it should. A restart of all of the API infrastructure servers globally was carried out. To help with the recovery, some of monitoring systems  which were triggering the bug were turned off. And as a result, a decision was made to restart servers gradually (at 6:49 PM), to avoid possible failures from restarting on a wide scale. By 7:07 PM, 45% of traffic was restored and 100% of traffic was routed to the API infrastructure at 7:10 PM.

# Corrective and preventative measures


Some of the measures that can be taken to improve or fix having this exact error incidence or similar incidences includes:
Always write test cases to confirm that programs are running as they ought to and hasn’t changed pre-existing functionality.
Disable the current configuration release mechanism until safer measures are implemented.
Implementation of verion control for easier and quicker rollback process.
