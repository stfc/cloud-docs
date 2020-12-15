=====================
LBaaS v2: Heat Stacks
=====================

Load balancing in a template consists of:
  - **Pool:** A group of servers which are identified by their floating IP addresses. The size of the pool can be increased or decreased in response to traffic to the instances detected by the load balancer.


  - **Listener:** Represent a listening endpoint for the vip. Listener resources listen for the client traffic.


  - **Health Monitor:** Monitors the health of the pool.


  - **Load Balancer:** Balances the traffic to server pools. In combination with the health monitor and listener, the load balancer can redistribute traffic to servers in the event of one server failing for example.

Octavia Resources in Heat
--------------------------

.. code-block:: bash

  pool:
    type: OS::Octavia::Pool
    properties:
      ## required ##########
      lb_algorithm: String #algorithm to distribute the load between the members of the pool: ROUND_ROBIN, LEAST_CONNECTIONS, SOURCE_IP
      protocol: String #protocol of the pool: TCP, HTTP, HTTPS, TERMINATED_HTTPS, SOURCE_IP

      ## optional ##########
      admin_state_up: Boolean #the administrative state of the pool
      description: String #description of the pool
      listener: String #listener name/ID to be associated with the pool
      loadbalancer: String #the loadbalancer name/ID to be associated to the pool
      name: String #name of this pool
      session_persistence: {"type": String, "cookie_name": String} #configuration of session persistence.
        #required - type: the method of session persistence feature: SOURCE_IP, HTTP_COOKIE, APP_COOKIE
        #optional - cookie_name: name of the cookie - required if type is APP_COOKIE
      tls_enabled: Boolean #default - false. Enable backend member re-encryption

  listener:
    type: OS::Octavia::Listener
    properties:
      ## required ############
      protocol: String # protocol on which to listen for client traffic: TCP, HTTP, HTTP/S, TERMINATED_HTTPS, PROXY, UDP
      protocol_port: Integer # TCP or UDP Port on which to listen for client traffic

      ## optional ############
      admin_state_up: Boolean
      allowed_cidrs: [String, String, ...]
      connection_limit: Integer
      default_pool: String
      default_tls_container_ref: String
      description: String
      loadbalancer: String
      name: String
      sni_container_refs: [Value, Value, ...]
      tenant_id: String

  health_monitor:
    type: OS::Octavia::HealthMonitor
    properties:
      ## required properties ############
      delay: Integer #(seconds) minimum time between regular connections of pool member
      max_retries: Integer # max number of permissible connection failures before changing member status to INACTIVE
      pool: String #name/ID of load balancing pool (type: octavia.pool)
      timeout: Integer # maximum number of seconds for a monitor to wait for a connection to be established before timeout.
      type: String #type of health monitor: PING, TCP, HTTP, HTTPS, UDP-CONNECT
      ## Optional ##############
      admin_state_up: Boolean #(default: true) administrative state of a health monitor.
      expected_codes: String #expected HTTP codes for a healthy monitor e.g. a single value (200), a list (200,202), or a range (202-204)
      http_method: String #method used for requests by HTTP monitor
      tenant_id: String #ID of tenant which owns the monitor
      url_path: String #HTTP path used in the HTTP request used by the monitor to test a member's health

  load_balancer:
    type: OS::Octavia::LoadBalancer
    properties:
      ## required ############
      vip_subnet: String #name/ID of the subnet on which to allocate the VIP address
      ## optional ############
      admin_state_up: Boolean #administrative state of the load balancer (default: true)
      description: String #description of the load balancer
      flavor: String #name/id of the flavor of the load balancer
      name: String #name of the loadbalancer
      provider: String #the provider of this load balancer
      tenant_id: String #id of the tenant which owns the loadbalancer
      vip_address: String #IP address for the VIP


The table below lists the attributes for each resource:

.. code-block:: bash

    | Resource                   	| Attributes
    |-----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | OS::Octavia::LoadBalancer  	| **flavor_id:** The flavor ID of the LoadBalancer<br><br>**pools:** the pools this LoadBalancer is associated with<br><br>**show:** detailed       information about the resource<br><br>**vip_address:** the VIP addresses of the LoadBalancer<br><br>**vip_port_id:** the VIP port of the LoadBalancer<br>           <br>**vip_subnet_id:** the VIP subnet of LoadBalancer 	|
    | OS::Octavia::Pool          	| **healthmonitor_id:** the ID of the health monitor associated with this pool<br><br>**listeners:** listener associated with this pool <br><br>**members:** members associated with this pool<br><br>**show:** detailed information about resource                                                                                                          	|
    | OS::Octavia::HealthMonitor 	| **pool:** the list of pools related to this monitor<br><br>**show:** detailed information about the resource                                                                                                                                                                                                                                               	|
    | OS::Octavia::Listener      	| **default_pool_id:** ID of the default pool the listener is associated to<br><br>**loadbalancers:** the ID of the load balancer this listener     is associated to<br><br>**show:** detailed information about resource                                                                                                                                        	|


Example
~~~~~~~
The example below shows how the Octavia resources can be defined in a Heat Template.


.. code-block:: yaml

  #HTTP health monitor
  health_monitor:
    type: OS::Octavia::HealthMonitor
    properties:
      delay: 3 #three second delay
      type: "HTTP"
      timeout: 3 # seconds
      max_retries: 3
      pool: {get_resource: pool}
      url_path: /healthcheck #this is a URL path that is configured on the servers in the pool that the monitor can reach to check pool health.

  pool:
    type: OS::Octavia::Pool
    properties:
      lb_algorithm: "LEAST_CONNECTIONS" #the preferred algorithm
      protocol: "HTTP"
      listener: {get_resource: listener}

  member:
    type: OS::Octavia::PoolMember
    properties:
      address: {get_attr: [server, first_address]}
      pool: {get_resource: pool}
      protocol_port: 80
      subnet: {get_param: private_subnet}

  listener:
    type: OS::Octavia::Listener
    properties:
      protocol: "HTTP"
      protocol_port: 80 # listen on the HTTP port
      loadbalancer: {get_resource: lb}

  lb:
    #define the load balancer and the private subnet to use
    type: OS::Octavia::LoadBalancer
    properties:
      vip_subnet: <private-subnet-id>

  # Attach a floating IP to the load balancer
  floating_ip_association:
    # Associate a floating IP to the Load Balancer so that it can be accessed
    # using an external IP
    type: OS::Neutron::FloatingIPAssociation
    properties:
      floatingip_id: <floating-ip-id>
      port_id: {get_attr: [lb,vip_port_id]}



References
----------

https://ibm-blue-box-help.github.io/help-documentation/heat/autoscaling-with-heat/

https://docs.openstack.org/heat/latest/template_guide/openstack.html

https://docs.openstack.org/octavia/train/reference/introduction.html

https://docs.openstack.org/octavia/train/user/guides/basic-cookbook.html
