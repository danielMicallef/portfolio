---
layout: ../../layouts/BlogPost.astro
draft: true
title: "Deploying a Django application"
lead: ""
date: 2025-10-07
tags: ["Django", "Deployment", "DevOps"]
---

## Simple deployment for small apps:

1. Setup WSGI
2. Configure gunicorn
3. Configure Nginx or Traeffik (for automatic ssl certificates)
    3.1. Steps to create automatic SSL certificates in nginx
4. Deployment process. The following needs to be done via CI/CD pipline like Github actions
    - Build the Images
    - Push Images to docker hub or similar
    - SSH to the deployment machine and pull the new images.
    - Restart the service. On service restart, any new migrations are run automatically. !Make sure migrations are not long running!
5. Setting behind Cloudflare. Assets are rendered via CDN

## Handling larger workloads

If you're lucky enough to have a successful application, and you start to realize that the worker is not enough, it may be time to scale.

1. Identify the bottleneck
Many times the first bottleneck in an application is the database. Improve queries, Add indexes, consider denormalization of the database, add caching, partitioning and more. Consider reading the blog about [database scaling strategies](/2024/12/10/strategies-scaling-database/)

2. If the bottleneck is the server's ability to handle the amount of requests/second, you will need a load balancer:
- Either use Nginx as a load balancer
- Use cloud based services (like AWS Elastic Load Balancing, or GCP Load Balancing)

### Nginx Instance
- __Small to Medium Scale Applications:__ If you're running a single application on one or a few servers in a single location, Nginx is often perfectly sufficient. It can handle thousands of requests per second with proper configuration.
- __Development and Testing:__ It's easy to set up locally or in a development environment for testing your application's architecture.
- __Cost-Sensitive Projects:__ The open-source version is free. You only pay for the infrastructure it runs on.
- __When You Need High Customization:__ You have full control over the configuration and can use a vast ecosystem of third-party modules to add specific functionalities.
- __Layer 7 (Application) Load Balancing:__ Nginx excels at inspecting HTTP/HTTPS traffic and making routing decisions based on URLs, headers, cookies, etc.

__Limitations of a self-managed Nginx setup:__

- __Single Point of Failure:__ A single Nginx instance is a single point of failure. You need to set up a high-availability (HA) pair (e.g., active-passive with `keepalived`) to mitigate this, which adds complexity.
- __Manual Scaling:__ If your traffic spikes, you have to manually scale the Nginx instances and the backend servers.
- __Maintenance Overhead:__ You are responsible for server provisioning, security patching, software updates, and monitoring.


## AWS and GCP Load Balancing Service
### AWS and GCP Load Balancing Services

Cloud providers offer load balancing as a fully managed service. They are designed for high availability, scalability, and tight integration with the rest of the cloud ecosystem.

__At what point should you use a cloud load balancer?__

You should seriously consider moving to a managed cloud load balancer when you hit one or more of the following points:

1. __High Availability and Fault Tolerance is Critical:__

   - Cloud load balancers are inherently highly available and fault-tolerant. They are distributed across multiple availability zones, so if one zone goes down, your traffic is automatically routed to healthy targets in other zones without any manual intervention.

2. __You Need Automatic Scaling:__

   - This is a major driver. When your application is behind a cloud load balancer integrated with an auto-scaling group (e.g., AWS ASG or GCP Managed Instance Group), the system can automatically add or remove backend servers based on traffic load or other metrics. This is very difficult to achieve smoothly with a self-managed Nginx setup.

3. __Your Traffic is High or Unpredictable:__

   - Managed load balancers are built to handle massive, sudden traffic spikes without breaking a sweat. They scale their own capacity automatically behind the scenes.

4. __You Are Operating in Multiple Geographic Regions:__

   - Cloud providers offer global load balancing solutions (e.g., Google Cloud Global External HTTPS Load Balancer) that can direct users to the closest application backend with a single IP address. This is complex and expensive to build yourself.

5. __You Want to Reduce Operational Overhead:__

   - If you want your team to focus on building application features instead of managing infrastructure, a managed service is the way to go. No more patching, OS updates, or configuring HA pairs for the load balancer itself.

6. __You Need Seamless Integration with Other Managed Services:__

   - Cloud LBs integrate easily with services like:

     - __Managed SSL/TLS Certificates:__ Free, auto-renewing certificates (e.g., AWS Certificate Manager).
     - __Web Application Firewall (WAF):__ Protect against common web exploits.
     - __CDN (e.g., CloudFront/Cloudflare):__ For caching content closer to users.
     - __Advanced Health Checks and Monitoring:__ Deeply integrated with the platform's monitoring tools.

### Summary

| Feature | Nginx (Self-Managed) | AWS/GCP Load Balancer |
| :--- | :--- | :--- |
| Scale | Small to Medium | Large to Global |
| Management | Manual (setup, patching, scaling) | Fully Managed |
| High Availability | Complex to set up (e.g., keepalived) | Built-in by default |
| Scalability | Manual | Automatic and seamless |
| Cost | Free software, pay for servers + ops time | Pay-as-you-go for traffic/usage |
| Integration | Manual integration with other tools | Deep integration with cloud ecosystem |