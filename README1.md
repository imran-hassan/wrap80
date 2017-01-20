
#1. Introduction

The contrail BGP implementation was designed from scratch to run on modern </br> 
server environments. The main goals were to be able to take advantage of </br> 
multicore CPUs, large (>4G) memory footprints and modern software development </br> 
techniques. </br>

BGP can be divided in the following components: </br>

1. **Input processing**: decoding and validating messages received from each  </br>
peer.
2. **Routing table operations**: modifying the routing table and determining  </br>
the set of updates to generate.
3. **Update encoding**: draining the update queue and encoding messages to a </br> 
set of peers.
4. **Management operations**: configuration processing and diagnostics. </br>

This blueprint provides a detailed description on defining a new origin field  </br>
by:

1. Making changes in Contrail configuration files. </br>
2. Making changes in Contrail GUI. </br>
3. Making changes in controller. </br>

All of these steps are to be performed for the new functionality to work  </br>
successfully.

#2. Problem statement </br>
###Normalize route origin when learning routes from a VM/VNF. </br>
This feature request is related to BGP as a Service (i.e. the vRouter peering  </br>
with a VNF running in a VM). It concerns the route origin field in BGP. On  </br>
PNF routers it is possible to override the route origin field of incoming  </br>
routes that are learned from peers. This same capability is needed when  </br>
Contrail learns routes from VNFs. In particular, this is related to the vLB  </br>
VNF because VNF does NOT want route origin to be part of the tiebreaker  </br>
protocol when choosing between routes. It is necessary to be able to set the  </br>
route origin field of routes learned from vLB VNFs to a single value across  </br>
entire network so that differences in route origin value between different  </br>
vLB instances won't contribute to route selection. </br>

#3. Proposed solution
Contrail by default exposes certain configurable options to the admin in  </br>
management console which are eventually used by underlying service when  </br>
making certain decisions or creating packets. In order to make origin field  </br>
configurable, following set of changes are needed: </br>

+ Expose a configurable option (Origin) in BGPaaS admin UI </br>

##3.1 Alternatives considered </br>
Describe pros and cons of alternatives considered. </br>

##3.2 User workflow impact </br>

Contrail GUI allows the user define a new route origin with multiple options.  </br>
User can click advanced options in Create to view the BGP Origin field. It  </br>
has four options: IGP, EGP, INCOMPLETE or NONE to be selected by the user. </br>

##3.3 UI changes </br>

Details in section 4.1 below. </br>

##3.4 Notification impact </br>

There were no changes made in logs, UVEs or alarms. </br>

#4. Implementation </br>
##4.1  Work items </br>

It has 4 modules. The GUI changes are mentioned below. </br>

The rest are defined in contrail-controller repo README.md </br>

###4.1.1 UI changes </br>

These steps are to be followed to make changes in contrail GUI to reflect the  </br>
impact of modifications in schema: </br>

+ Declare bgp_origin as optional field in file  </br>
**webroot/common/api/jsonDiff.helper.js** in optFields. </br>

+ Declare bgp_origin format in  </br>
**webroot/config/services/bgpasaservice/ui/js/bgpAsAServiceFormatter.js**,  </br>
which is called by frontend to show appropriate bgp origin value. </br>

+ In ContrailConfigModel, add bgp_origin in defaultConfig which is present in  </br>
this file: 
**webroot/config/services/bgpasaservice/ui/js/models/bgpAsAServiceModel.js**. </br>

String value from frontend is converted into integer value to be sent to the  </br>
backend.
Then the value received at frontend is validated that whether it is 0, 1, 2  </br>
or 3.3

+ To add a new field on the GUI, the structure of a drop down menu is defined  </br>
in 
**webroot/config/services/bgpasaservice/ui/js/views/bgpAsAServiceEditView.js**. </br>
 This enables different options i.e. IGP, BGP, ICOMPLETE and NONE to be  </br>
visible in the drop down menu at Edit View on GUI.

+ To make the BGP origin value visible in the Grid View, changes are made in  </br>
**webroot/config/services/bgpasaservice/ui/js/views/bgpAsAServiceGridView.js**. </br>

+ The value of BGP origin is bonded with BGP origin formatter by making  </br>
changes in “this.bgpOrigunViewFormattter” function. </br>

By making the above mentioned changes, the BGP Origin Field will become </br> 
configurable in the UI.
On frontend, we get field of **BGP origin** in the tabs **Create**, **Edit**  </br>
and **Grid view**. BGP origin field is also visible in the tab “BGP as a  </br>
service.”

![alt text](images/sec_4.1.1_j.png "Img 10") </br>

![alt 
text](https://github.com/saad-ngnware/test-repo/blob/master/images/sec_4.1.1_k.
png "Img 11") </br>

![alt 
text](https://github.com/saad-ngnware/test-repo/blob/master/images/sec_4.1.1_l.
png "Img 12") </br>

An object is passed from frontend to API Server when we create BGP as a  </br>
service.

Details in contrail-controller repo README.md </br>

#5. Performance and scaling impact </br>
##5.1 API and control plane </br>

There are no changes in scalability of API and Control Plane. </br>
##5.2 Forwarding performance </br>
We do not expect any change to the forwarding performance. </br>

#6. Upgrade </br>
The BGP origin field is a new field and hence does not have any upgrade impact. </br>

#7. Deprecations </br>
There are no deprecations when this change is made. </br>

#8. Dependencies </br>
There are no dependencies for this feature. </br>

#9. Testing </br>
##9.1 Unit test </br>

GUI unit test: Check if values are visible on frontend and are passed to the  </br>
backend.

##9.2 Dev test </br>

Flow Test Steps: </br>

+ Check if value of BGP origin is received from frontend. </br>

These tests were completed successfully. </br>

#10. Documentation Impact </br>
BGP origin field details have to be added in user documentation. </br>

#11. References </br>
[bgp_design](http://juniper.github.io/contrail-vnc/bgp_design.html) </br>

[adding-bgp-knob-to-opencontrail](http://www.opencontrail.org/adding-bgp-knob-t
o-opencontrail/) </br>

[contrail-controller 
(source-code)](https://github.com/Juniper/contrail-controller/tree/master/src/v
nsw/agent) </br>
