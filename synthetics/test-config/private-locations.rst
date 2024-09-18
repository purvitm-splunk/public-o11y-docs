.. _private-locations:

*****************
Private locations
*****************

.. meta::
    :description: Run synthetic tests from an internal site or private web application to quickly find defects using Splunk Synthetic Monitoring. 

A private location is a software package that offers a quick and easy deployment of Splunk Synthetic Monitoring solutions beyond the public network so that you can find, fix, and prevent web performance defects on any internal web application, in any environment - whether inside or outside of your firewalls. Private locations allow Splunk Synthetics Monitoring users to test sooner in the development cycle and against internal sites or applications that aren't available to the public.

Customers can, through the Splunk Synthetics Monitoring web interface, create new private locations and open a runner to run any checks assigned to them.

What is a runner?
===================

A runner is a Docker container set up to run tests from a particular private location. A single private location can have one or more runners. 

A location consists of a queue of tests assigned to a particular private location. Runners pick up runs from the queue, so the more active runners you have, the faster the queue of tests is processed. 

Splunk Synthetic Monitoring doesn't track how many runners there are for a given location. It is up to you to manage your own fleet of runners. 


Use cases for private locations
=================================

* Test private applications that aren't exposed to the public.
* Test pre-production applications which don't have public staging sites.
* Gain a higher level of flexibility in giving Splunk Synthetic Monitoring access to applications.
* Test from locations not currently supported by Splunk Synthetic Monitoring's public locations.


Requirements 
=============


.. list-table::
  :header-rows: 1
  :widths: 20 80 

  * - :strong:`Requirement`
    - :strong:`Description`
  * - Docker
    - 
        * The Docker container requires the host have the ifb kernel module installed. 
        * The Docker container needs outbound internet access; however it doesn't need inbound access.  
  * - Allowlist
    - 
        * ``runner.<realm>.signalfx.com`` 
        * ``*.signalfx.com`` 
        * ``*.amazonaws.com``
        * ``quay.io/signalfx``
        * ``quay.io/v2``
  * - Operating system   
    -  Linux, Windows, or macOS


For optimal performance when running Browser tests:

* Linux
* 2.3 GHz Dual-Core Intel Xeon (or equivalent) processor
* 8 GB RAM, 2 cores


Set up a new private location
===============================

Follow these steps to set up a new private location:

1. In Splunk Synthetic Monitoring, select the settings gear icon, then :guilabel:`Private locations`.  
2. Select :guilabel:`+ Add` and add a name. 
3. Follow the steps in the guided setup to set up your runner. 
4. Save your private location. 


What you can do with your private location ID 
------------------------------------------------------------

Each private location has a corresponding private location ID. With this ID, you can:

* Build charts or dashboards
* Search for metrics by private location
* Refer to your private location ID if you're interacting with the Splunk Synthetics Monitoring APIs. 

Manage your tokens
--------------------
It is your responsibility to update and manage your tokens. Tokens are valid for one year. For added security, create a secret environment variable for your token in Docker. Consider creating a second token to provide coverage before your first token expires. You are not notified of expiring tokens.


Working with Docker 
======================================
Here is some guidance for working with Docker. 

Limit logging in Docker 
------------------------------------

Follow these steps to limit logging:

#. Create a file in a directory like this: ``/etc/docker/daemon.json``.

#. In the file, add: 

.. code:: yaml

    {
      "log-driver": "local",
      "log-opts": {
        "max-size": "10m",
        "max-file": "3"
      }
    }

#. Restart your docker service: ``sudo systemctl docker.service restart``.



Add certificates in Synthetics
------------------------------------------------------
Splunk Synthetic Monitoring supports injecting custom root CA certificates for API and Uptime tests running from your private locations. Client keys and certificates aren't supported at this time. 

#. Create a folder called ``certs`` on your host machine and place the CA Certificate (in CRT format) in the folder.

#. Add the certs folder as a volume to the container ``(-v ./certs:/usr/local/share/ca-certificates/my_certs/)``.

#. Modify the command you use when launching the container to update the CA Certificate cache before starting the agent binary ``(bash -c "sudo update-ca-certificates && bundle exec bin/start_runner)``.


For example, here is what a command might look like after you modify it to fit your environment:  

.. code:: yaml

    docker run -e "RUNNER_TOKEN=<insert-token>" --volume=`pwd`/certs:/usr/local/share/ca-certificates/my_certs/ quay.io/signalfx/splunk-synthetics-runner:latest bash -c "sudo update-ca-certificates && bundle exec bin/start_runner"


.. Note:: Custom root CA certificates aren't supported for Browser tests. Browser tests require SSL/TLS validation for accurate testing. Optionally, you can deactivate SSL/TLS validation for Browser tests when necessary.






Configuring Proxy Settings for Private Locations
===================================================

In environments where direct internet access is restricted, you can route synthetic test traffic through a proxy server by configuring the following environment variables:

* HTTP_PROXY: Specifies the proxy server for HTTP traffic.

    * Example: export HTTP_PROXY="\http://proxy.example.com:8080"

* HTTPS_PROXY: Specifies the proxy server for HTTPS traffic.

    * Example: export HTTPS_PROXY="\https://proxy.example.com:8443"

* NO_PROXY: Specifies a comma-separated list of domains or IP addresses that should bypass the proxy.

    * Example: export NO_PROXY="localhost,127.0.0.1,.internal-domain.com"

For example, here is what a command might look like after you modify it to fit your environment:


.. code:: yaml

    docker run --cap-add NET_ADMIN -e "RUNNER_TOKEN=*****" quay.io/signalfx/splunk-synthetics-runner:latest -e NO_PROXY=".signalfx.com,.amazonaws.com"  -e HTTPS_PROXY="https://172.17.0.1:1234" -e HTTP_PROXY="http://172.17.0.1:1234"
    
In this example:

HTTP_PROXY and HTTPS_PROXY are set to route traffic through a proxy at \http://172.17.0.1:1234.

NO_PROXY is configured to bypass the proxy for local addresses and specific domains like .signalfx.com and .amazonaws.com.

Ensure that these variables are correctly configured to comply with your network policies. This setup allows the synthetic tests to communicate securely and efficiently in a controlled network environment.







Assess the health of your private location
==============================================

A private location's health depends on three factors:

.. list-table::
  :header-rows: 1
  :widths: 20 40 40 

  * - :strong:`Factor`
    - :strong:`Description`
    - :strong:`Solution`
  * - Active runner
    - At least one runner is actively checking in.
    - If no runners are checking in, set up new runners for the private location. 
  * - Used in tests
    - The private location is currently being used in one or more tests.
    - If you need to delete a private location, you need to first delete it from all tests.
  * - Clear queue
    - The queue for a given location is being cleared periodically and is not backed up.
    - If the queue is backed up, add new runners to the private location.

Troubleshoot queue length and latency
---------------------------------------------------

If both the queue latency and length increase over time, then add more runners to improve performance. 

If your queue latency increases but your queue length doesn't, then try these troubleshooting methods:

* Check to see if a step is delaying the rest of the test
* Investigate whether you have the sufficient resources to run private location runners on your machines.

The maximum number of runs in a queue is 100,000. 

Any runs older than one hour are removed from the queue. 






Automatically Update Private Location Runners - Using Watchtower
================================================================

Synthetics will often publish updates to our private location docker images and it is critical that your organization automates the upgrade of your locations.

Watchtower is entirely optional, but we do require that you have an auto-upgrade solution in place for the private location runner. Failing to automatically update our Docker container will result in inconsistent data and loss of functionality.

`Watchtower <https://github.com/v2tec/watchtower>`_
is a 3rd party open source Docker container that will connect to remote Docker repositories on a schedule and check for updates. When an updated image is found Watchtower will instruct your Docker host to pull the newest image from the repository, stop the container, and start it again. It will ensure that environment variables, network settings, and links between containers are intact. 

On your Docker host, you can simply launch the Watchtower container via command line:

.. code:: yaml

  docker run -d \
    --name watchtower \
    -v /var/run/docker.sock:/var/run/docker.sock \
    v2tec/watchtower --label-enable --cleanup

.. Note:: Using the "label-enable" flag will ensure that only containers with the correct label, like the private location runner, will be auto-updated.

To apply this in practice, you'd label the specific container you want Watchtower to update when you create or run it. For example:

.. code:: yaml

  docker run -d \
    --name private-location-runner \
    --label=com.centurylinklabs.watchtower.enable=true \
    private-location/runner-image:latest

In this case, Watchtower will only monitor and update the private-location-runner container because of the label com.centurylinklabs.watchtower.enable=true.

There are additional options available in the `Watchtower documentation <https://github.com/v2tec/watchtower#options>`_ that you should explore, including auto-cleanup of old images to ensure that your Docker host does not hold on to outdated images.

It is important to note that in order for Watchtower to issue commands to the Docker host, it requires the docker.sock volume or TCP connection and this provides Watchtower with full administrative access to your Docker host. This level of access should not be taken lightly and it is one of the reasons private-location decided to separate the auto-update procedure from the private-location agent. If you are uncomfortable with providing Watchtower with this level of access you are encouraged to explore other options.
