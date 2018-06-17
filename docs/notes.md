Notes
=====

As the Pet Clinic application runs, its application server creates log files that gradually consume disk space. If you find that disk space on your host is getting low, you may execute the following command to delete all but the last 7 days' logs: 

    $ find ./logs \( -name '*.log' -o -name '*.txt' \) -type f -mtime +7 -exec rm -f {} \;

[Some Linux distributions have a bug](https://github.com/moby/moby/issues/3182#issuecomment-256532928) that causes Docker not to release disk space when containers and images are removed. If deleting log files does not free enough space, you may stop the Docker service, delete its files, and restart the service. _This will delete all Docker containers on the host!_ 

    $ sudo service docker stop
    $ sudo rm -rf /var/lib/docker
    $ sudo service docker start

You may recreate the MySQL and Apache Tomcat containers by re-running the `docker run` commands (steps 5 and 6) in the [previous section](/deploying-petclinic/).