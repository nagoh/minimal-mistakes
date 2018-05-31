---
classes: wide
published: false
---
Recently, i've had to setup a Teamcity server within Azure. I've found to do this poorly is really easy 😜, but to do a job I'd class as ready for the SME/Enterprise is not so straightforward. 

These are some things I'm hoping to get from this setup
1. A private deployment (i.e. accessible only by VPN)
2. HTTPS enabled
3. TeamCity authentication integrated to Azure Active Directory 

I'll run you through each of these points in a bit of detail

## Private Deployment

For illustrative purposes I'll use a really simple network design:
* **Virtual Network** vnet-build
* **Address Space** 10.4.0.0/16
* *Subnets*
    - **vm-subnet** 10.4.0.0/24
    - **ag-subnet** 10.4.1.0/24


![alt]({{ site.url }}{{ site.baseurl }}/assets/images/502.png)