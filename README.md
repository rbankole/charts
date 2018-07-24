
<!DOCTYPE html>
<html>
  <head>
    <title>Getting started with the ACS Engine to deploy Kubernetes in Azure – DevOps Automation & Monitoring – ¯\_(ツ)_/¯</title>

        <meta charset="utf-8" />
    <meta content='text/html; charset=utf-8' http-equiv='Content-Type'>
    <meta http-equiv='X-UA-Compatible' content='IE=edge'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0, maximum-scale=1.0'>

    
    <meta name="description" content="Over the past 6 months, I have had to use the Azure Container Service Engine to deploy and maintain K8s Clusters in Azure running both Linux and Windows Nodes in the same Cluster. This type of configuration in Azure is currently only possible using the ACS Engine. First time users of the ACS Engine may find the process incredibly daunting as it is the complete opposite experience of deploying a K8s Cluster using acs or aks in the Azure CLI; instead of having everything managed for you, you are responsible for managing the configuration and deployment of the Cluster. As such you are able to configure almost every aspect of your K8s Cluster before deploying it.

" />
    <meta property="og:description" content="Over the past 6 months, I have had to use the Azure Container Service Engine to deploy and maintain K8s Clusters in Azure running both Linux and Windows Nodes in the same Cluster. This type of configuration in Azure is currently only possible using the ACS Engine. First time users of the ACS Engine may find the process incredibly daunting as it is the complete opposite experience of deploying a K8s Cluster using acs or aks in the Azure CLI; instead of having everything managed for you, you are responsible for managing the configuration and deployment of the Cluster. As such you are able to configure almost every aspect of your K8s Cluster before deploying it.

" />
    
    <meta name="author" content="DevOps Automation & Monitoring" />

    
    <meta property="og:title" content="Getting started with the ACS Engine to deploy Kubernetes in Azure" />
    <meta property="twitter:title" content="Getting started with the ACS Engine to deploy Kubernetes in Azure" />
    

    <!--[if lt IE 9]>
      <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
    <![endif]-->

    <link rel="stylesheet" type="text/css" href="/style.css" />
    <link rel="alternate" type="application/rss+xml" title="DevOps Automation & Monitoring - ¯\_(ツ)_/¯" href="/feed.xml" />

    <!-- Created with Jekyll Now - http://github.com/barryclark/jekyll-now -->
  </head>

  <body>
    <div class="wrapper-masthead">
      <div class="container">
        <header class="masthead clearfix">
          <a href="/" class="site-avatar"><img src="https://raw.githubusercontent.com/starkfell/starkfell.github.io/master/images/salad.png" /></a>

          <div class="site-info">
            <h1 class="site-name"><a href="/">DevOps Automation & Monitoring</a></h1>
            <p class="site-description">¯\_(ツ)_/¯</p>
          </div>

          <nav>
            <a href="/">Blog</a>
            <a href="/about">About</a>
          </nav>
        </header>
      </div>
    </div>

    <div id="main" role="main" class="container">
      <article class="post">
  <h1>Getting started with the ACS Engine to deploy Kubernetes in Azure</h1>

  <div class="entry">
    <p>Over the past 6 months, I have had to use the <strong><a href="https://github.com/Azure/acs-engine">Azure Container Service Engine</a></strong> to deploy and maintain K8s Clusters in Azure running both Linux and Windows Nodes in the same Cluster. This type of configuration in Azure is currently only possible using the ACS Engine. First time users of the ACS Engine may find the process incredibly daunting as it is the complete opposite experience of deploying a K8s Cluster using acs or aks in the Azure CLI; instead of having everything managed for you, you are responsible for managing the configuration and deployment of the Cluster. As such you are able to configure almost every aspect of your K8s Cluster before deploying it.</p>

<p>Because the learning curve of the ACS Engine can be quite steep, I wanted to provide a reference guide allowing other individuals a quicker way to get started from scratch as well having it for future reference for myself. For the complete documentation on the Azure Container Service Engine, make sure to review the <strong><a href="https://github.com/Azure/acs-engine/tree/master/docs">Official Documenation</a></strong>.</p>

<p>More posts will be coming in the near future detailing some of the customization options available to you in the Azure Container Service Engine.</p>

<h1 id="overview">Overview</h1>

<p>This article covers the basics of deploying a new K8s Cluster in Azure using the following steps and the acs-engine. These instructions were written for and tested on Ubuntu 16.04 using a standard linux user starting in their home directory. The instructions <em>should</em> work on Bash on Ubuntu for Windows but haven’t been tested.</p>

<ul>
  <li>Installing Azure CLI 2.0</li>
  <li>Instll the latest version of kubectl</li>
  <li>Installing the ACS Engine</li>
  <li>Generating an SSH Key</li>
  <li>Create a Service Principal in the Azure Subscription</li>
  <li>Create a Cluster Definition File</li>
  <li>Create a new Resource Group for the Kubernetes Cluster</li>
  <li>Deploy the Kubernetes ARM Template to the Resource Group</li>
  <li>Connect to the Kubernetes Cluster</li>
</ul>

<h2 id="prerequisites">Prerequisites</h2>

<ul>
  <li>Access to an existing Azure Subscription and Administrative Rights to the Subscription</li>
  <li>A Linux VM with the Azure CLI Installed</li>
  <li><strong>curl</strong> is required to be installed and <strong>vim</strong> is highly recommended</li>
  <li>5 to 10 CPU Cores available in your Azure Subscription for Standard_D2_v2 VMs</li>
  <li>The Azure Subscription ID used in the documentation below, <strong>d5b31b94-d91c-4ef8-b9d0-30193e6308ee</strong>, needs to be replaced with your Azure Subscription ID.</li>
</ul>

<p>The Name of the Service Principal and DNS Prefix for the documentation below is <strong>azure-k8s-dev</strong>.</p>

<h2 id="installing-azure-cli-20">Installing Azure CLI 2.0</h2>

<p>Run the following command to install Azure CLI 2.0.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">echo</span> <span class="s2">"deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main"</span> | <span class="nb">sudo </span>tee /etc/apt/sources.list.d/azure-cli.list <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt-key adv <span class="nt">--keyserver</span> packages.microsoft.com <span class="nt">--recv-keys</span> 417A0893 <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt-get install <span class="nt">-y</span> apt-transport-https <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>apt-get update <span class="o">&amp;&amp;</span> <span class="nb">sudo </span>apt-get install <span class="nt">-y</span> azure-cli
</code></pre></div></div>

<h2 id="install-the-latest-version-of-kubectl">Install the latest version of kubectl</h2>

<p>Run the following command to install the latest version of <strong>kubectl</strong>.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>curl <span class="nt">-LO</span> https://storage.googleapis.com/kubernetes-release/release/<span class="k">$(</span>curl <span class="nt">-s</span> https://storage.googleapis.com/kubernetes-release/release/stable.txt<span class="k">)</span>/bin/linux/amd64/kubectl <span class="o">&amp;&amp;</span> <span class="se">\</span>
chmod +x ./kubectl <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>mv ./kubectl /usr/local/bin/kubectl
</code></pre></div></div>

<h2 id="install-the-acs-engine">Install the ACS Engine</h2>

<p>If you want to install the <strong><a href="https://github.com/Azure/acs-engine/releases/latest">latest</a></strong> version of the acs-engine, which at the time of this writing is <strong>v0.13.0</strong>, run the following command.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://github.com/Azure/acs-engine/releases/download/v0.13.0/acs-engine-v0.13.0-linux-amd64.tar.gz <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">tar</span> <span class="nt">-xzvf</span> acs-engine-v0.13.0-linux-amd64.tar.gz <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>cp acs-engine-v0.13.0-linux-amd64/acs-engine /usr/bin/acs-engine <span class="o">&amp;&amp;</span> <span class="se">\</span>
<span class="nb">sudo </span>cp acs-engine-v0.13.0-linux-amd64/acs-engine /usr/local/bin/acs-engine
</code></pre></div></div>

<p>If you want to install a particular version of the acs-engine, visit <strong>https://github.com/Azure/acs-engine/tags</strong>.</p>

<h2 id="generate-an-ssh-key">Generate an SSH Key</h2>

<p>Run the command below to generate an SSH Key using <strong>ssh-keygen</strong>. The Name of the SSH Key is named after the Service Principal and DNS Prefix being used in this walkthrough, <strong>azure-k8s-dev</strong>.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>ssh-keygen <span class="nt">-t</span> rsa <span class="nt">-b</span> 2048 <span class="nt">-C</span> <span class="s2">"azure-k8s-dev-access-key"</span> <span class="nt">-f</span> ~/.ssh/azure-k8s-dev-access-key <span class="nt">-N</span> <span class="s1">''</span>
</code></pre></div></div>

<h2 id="create-a-service-principal-in-the-azure-subscription">Create a Service Principal in the Azure Subscription</h2>

<p>Run the commands below using the Azure CLI.</p>

<p>Login to your Azure Subscription.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>az login <span class="nt">-u</span> account.name@microsoft.com
</code></pre></div></div>

<p>Set the Azure Subscription you want to work with.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>az account <span class="nb">set</span> <span class="nt">-s</span> d5b31b94-d91c-4ef8-b9d0-30193e6308ee
</code></pre></div></div>

<p>Run the following command create a Service Principal in the Azure Subscription.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>    az ad sp create-for-rbac <span class="se">\</span>
    <span class="nt">--role</span><span class="o">=</span><span class="s2">"Contributor"</span> <span class="se">\</span>
    <span class="nt">--name</span><span class="o">=</span><span class="s2">"azure-k8s-dev"</span> <span class="se">\</span>
    <span class="nt">--password</span><span class="o">=</span><span class="s2">"UseAzureKeyVault1!"</span> <span class="se">\</span>
    <span class="nt">--scopes</span><span class="o">=</span><span class="s2">"/subscriptions/d5b31b94-d91c-4ef8-b9d0-30193e6308ee"</span>
</code></pre></div></div>

<p>You should get a similar response back after a few seconds. Additionally, the App will appear in the <strong>App Registrations</strong> section in the <strong><a href="https://portal.azure.com">Azure Portal</a></strong>.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Retrying role assignment creation: 1/36
Retrying role assignment creation: 2/36
<span class="o">{</span>
  <span class="s2">"appId"</span>: <span class="s2">"fc045a69-cc77-4331-aa07-e70b682a414e"</span>,
  <span class="s2">"displayName"</span>: <span class="s2">"azure-k8s-dev"</span>,
  <span class="s2">"name"</span>: <span class="s2">"http://azure-k8s-dev"</span>,
  <span class="s2">"password"</span>: <span class="s2">"UseAzureKeyVault1!"</span>,
  <span class="s2">"tenant"</span>: <span class="s2">"b7ede2be-6495-48d3-ace8-24d68a53cf2d"</span>
<span class="o">}</span>
</code></pre></div></div>

<p>Make note of the the <strong>AppId</strong> as it will be used for the <strong>clientId</strong> field in the Cluster Definition File in the next section.</p>

<h2 id="create-a-cluster-definition-file">Create a Cluster Definition File</h2>

<p>Several Cluster Definition File examples can be found in the <strong><a href="https://github.com/Azure/acs-engine/tree/master/examples">ACS Engine GitHub Repository</a></strong>. The Cluster Definition File that we will be using here will be a modified version of an existing example.</p>

<p>The acs-engine Cluster Definition Files are JSON files that allow you to configure several options about your K8s Cluster. Below are the some of the more common options you will modify.</p>

<div class="language-text highlighter-rouge"><div class="highlight"><pre class="highlight"><code>orchestratorType         - Kubernetes (Other options include Swarm, Swarm Mode, and DCOS).
orchestratorVersion      - The version of Kubernetes to deploy, i.e. - 1.6.1, 1.7.2, 1.8.2, 1.9.1, 1.9.3.
masterProfile            - The number of Master Nodes to deploy, the DNS Prefix to use, VM Size, type of Storage to use, OS Disk Size (GB).
agentPoolProfiles        - The name of the pool, number of Nodes to deploy, VM Size, type of Storage to use, OS Disk Size (GB), Availability Set Profile, OS Type.
linuxProfile             - the admin Username and SSH Key used to access the Linux Nodes.
windowsProfile           - the admin Username and Password used to access the Windows Nodes.
servicePrincipalProfile  - The Service Principal Client ID and Service Principal Password.
</code></pre></div></div>

<p>For the purposes of this walkthrough, we are going to deploy a <strong>vanilla</strong> Deployment of Kubernetes 1.9.3 using the following Cluster Definition File. Copy and paste the contents below into a file called <strong>deploy-k8s-1.9.3.json</strong>.</p>

<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">{</span><span class="w">
  </span><span class="s2">"apiVersion"</span><span class="p">:</span><span class="w"> </span><span class="s2">"vlabs"</span><span class="p">,</span><span class="w">
  </span><span class="s2">"properties"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="s2">"orchestratorProfile"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="s2">"orchestratorType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Kubernetes"</span><span class="p">,</span><span class="w">
      </span><span class="s2">"orchestratorRelease"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.9"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="s2">"masterProfile"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="s2">"count"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
      </span><span class="s2">"dnsPrefix"</span><span class="p">:</span><span class="w"> </span><span class="s2">""</span><span class="p">,</span><span class="w">
      </span><span class="s2">"vmSize"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Standard_D2_v2"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="s2">"agentPoolProfiles"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"linuxpool1"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"count"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="p">,</span><span class="w">
        </span><span class="s2">"vmSize"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Standard_D2_v2"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"availabilityProfile"</span><span class="p">:</span><span class="w"> </span><span class="s2">"AvailabilitySet"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">],</span><span class="w">
    </span><span class="s2">"linuxProfile"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="s2">"adminUsername"</span><span class="p">:</span><span class="w"> </span><span class="s2">"linuxadmin"</span><span class="p">,</span><span class="w">
      </span><span class="s2">"ssh"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="s2">"publicKeys"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
          </span><span class="p">{</span><span class="w">
            </span><span class="s2">"keyData"</span><span class="p">:</span><span class="w"> </span><span class="s2">""</span><span class="w">
          </span><span class="p">}</span><span class="w">
        </span><span class="p">]</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="s2">"servicePrincipalProfile"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="s2">"clientId"</span><span class="p">:</span><span class="w"> </span><span class="s2">""</span><span class="p">,</span><span class="w">
      </span><span class="s2">"secret"</span><span class="p">:</span><span class="w"> </span><span class="s2">""</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre></div></div>

<p>You’ll notice the following name-pair values in the definition above need to be filled in with their respective values.</p>

<div class="language-text highlighter-rouge"><div class="highlight"><pre class="highlight"><code>dnsPrefix = azure-k8s-dev
keyData   = {SSH_PUBLIC_KEY}
clientId  = appId
secret    = UseAzureKeyVault1!
</code></pre></div></div>

<p>Once you have added in the respective values of the name-pairs listed above, the <strong>deploy-k8s-1.9.3.json</strong> file should appear similar to what is shown below.</p>

<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">{</span><span class="w">
  </span><span class="s2">"apiVersion"</span><span class="p">:</span><span class="w"> </span><span class="s2">"vlabs"</span><span class="p">,</span><span class="w">
  </span><span class="s2">"properties"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="s2">"orchestratorProfile"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="s2">"orchestratorType"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Kubernetes"</span><span class="p">,</span><span class="w">
      </span><span class="s2">"orchestratorRelease"</span><span class="p">:</span><span class="w"> </span><span class="s2">"1.9"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="s2">"masterProfile"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="s2">"count"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
      </span><span class="s2">"dnsPrefix"</span><span class="p">:</span><span class="w"> </span><span class="s2">"azure-k8s-dev"</span><span class="p">,</span><span class="w">
      </span><span class="s2">"vmSize"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Standard_D2_v2"</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="s2">"agentPoolProfiles"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="s2">"name"</span><span class="p">:</span><span class="w"> </span><span class="s2">"linuxpool1"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"count"</span><span class="p">:</span><span class="w"> </span><span class="mi">2</span><span class="p">,</span><span class="w">
        </span><span class="s2">"vmSize"</span><span class="p">:</span><span class="w"> </span><span class="s2">"Standard_D2_v2"</span><span class="p">,</span><span class="w">
        </span><span class="s2">"availabilityProfile"</span><span class="p">:</span><span class="w"> </span><span class="s2">"AvailabilitySet"</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">],</span><span class="w">
    </span><span class="s2">"linuxProfile"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="s2">"adminUsername"</span><span class="p">:</span><span class="w"> </span><span class="s2">"linuxadmin"</span><span class="p">,</span><span class="w">
      </span><span class="s2">"ssh"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
        </span><span class="s2">"publicKeys"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
          </span><span class="p">{</span><span class="w">
            </span><span class="s2">"keyData"</span><span class="p">:</span><span class="w"> </span><span class="s2">"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQrDd2IyLZIYn02wrltHKwC3UXXkkO2Br1jiYdJ1yYA8Q2Wm7LJj4c2lnKD9c/VCtR2w5cBwTz9D/JGwFJtHEPUZXOq3CDxWcPRE8GfRK9f1OZlvFwuTHTEJaza8KRVRhrXX9Tjtl2a94R7uSXr7NIKFgopjGkJ9BgSlufh0lUiWoAg1/e7cNXi3tiewu6lI+bG1v5aKmgKfITpbe56YIBYNzQEnxjCQdIye5hafz3XoxVkGaKst072cByygERqFPV6QFcJ9CITMgL3SoI3/XTPdg+hKYFU2VL5Xc6Chi2q3WVM69IkxnGpZOES8nxWRfkEAX08zsWtjpVu18DlEm/ azure-k8s-dev-access-key"</span><span class="w">
          </span><span class="p">}</span><span class="w">
        </span><span class="p">]</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">},</span><span class="w">
    </span><span class="s2">"servicePrincipalProfile"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
      </span><span class="s2">"clientId"</span><span class="p">:</span><span class="w"> </span><span class="s2">"fc045a69-cc77-4331-aa07-e70b682a414e"</span><span class="p">,</span><span class="w">
      </span><span class="s2">"secret"</span><span class="p">:</span><span class="w"> </span><span class="s2">"UseAzureKeyVault1!"</span><span class="w">
    </span><span class="p">}</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre></div></div>

<p>Save your changes to the <strong>deploy-k8s-1.9.3.json</strong> file and close it.</p>

<p><em>Note: The full list of customizable options you can modify can be found in the <strong><a href="https://github.com/Azure/acs-engine/blob/master/docs/clusterdefinition.md">Cluster Definition Documentation</a></strong>.</em></p>

<h2 id="generate-the-deployment-arm-templates-using-the-acs-engine">Generate the deployment ARM Templates using the ACS Engine</h2>

<p>Run the following command to generate the deployment ARM Templates.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>acs-engine generate deploy-k8s-1.9.3.json
</code></pre></div></div>

<p>The ARM Templates will generated in a few seconds and you should see the following response.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>INFO[0000] Generating assets into _output/azure-k8s-dev...
</code></pre></div></div>

<p>The acs-engine generates the following folder structure based off of the <strong>dnsPrefix</strong> that we previously set in the <strong>masterProfile</strong> section of the Cluster Definition file. Shown below is what the folder structure when the DNS Prefix is set to <strong>azure-k8s-dev</strong>, as was done in the previous steps.</p>

<div class="language-text highlighter-rouge"><div class="highlight"><pre class="highlight"><code>_output/azure-k8s-dev --&gt; apimodel.json
_output/azure-k8s-dev --&gt; apiserver.crt
_output/azure-k8s-dev --&gt; apiserver.key
_output/azure-k8s-dev --&gt; azuredeploy.json
_output/azure-k8s-dev --&gt; azuredeploy.parameters.json
_output/azure-k8s-dev --&gt; ca.crt
_output/azure-k8s-dev --&gt; ca.key
_output/azure-k8s-dev --&gt; client.crt
_output/azure-k8s-dev --&gt; client.key
_output/azure-k8s-dev --&gt; kubectlClient.crt
_output/azure-k8s-dev --&gt; kubectlClient.key
_output/azure-k8s-dev --&gt; kubeconfig
                          kubeconfig --&gt; kubeconfig.australiaeast.json
                          kubeconfig --&gt; kubeconfig.australiasoutheast.json
                          kubeconfig --&gt; kubeconfig.brazilsouth.json
                          kubeconfig --&gt; kubeconfig.canadacentral.json
                          kubeconfig --&gt; kubeconfig.canadaeast.json
                          kubeconfig --&gt; kubeconfig.centralindia.json
                          kubeconfig --&gt; kubeconfig.centraluseuap.json
                          kubeconfig --&gt; kubeconfig.centralus.json
                          kubeconfig --&gt; kubeconfig.chinaeast.json
                          kubeconfig --&gt; kubeconfig.chinanorth.json
                          kubeconfig --&gt; kubeconfig.eastasia.json
                          kubeconfig --&gt; kubeconfig.eastus2euap.json
                          kubeconfig --&gt; kubeconfig.eastus2.json
                          kubeconfig --&gt; kubeconfig.eastus.json
                          kubeconfig --&gt; kubeconfig.germanycentral.json
                          kubeconfig --&gt; kubeconfig.germanynortheast.json
                          kubeconfig --&gt; kubeconfig.japaneast.json
                          kubeconfig --&gt; kubeconfig.japanwest.json
                          kubeconfig --&gt; kubeconfig.koreacentral.json
                          kubeconfig --&gt; kubeconfig.koreasouth.json
                          kubeconfig --&gt; kubeconfig.northcentralus.json
                          kubeconfig --&gt; kubeconfig.northeurope.json
                          kubeconfig --&gt; kubeconfig.southcentralus.json
                          kubeconfig --&gt; kubeconfig.southeastasia.json
                          kubeconfig --&gt; kubeconfig.southindia.json
                          kubeconfig --&gt; kubeconfig.uksouth.json
                          kubeconfig --&gt; kubeconfig.ukwest.json
                          kubeconfig --&gt; kubeconfig.usgoviowa.json
                          kubeconfig --&gt; kubeconfig.usgovvirginia.json
                          kubeconfig --&gt; kubeconfig.westcentralus.json
                          kubeconfig --&gt; kubeconfig.westeurope.json
                          kubeconfig --&gt; kubeconfig.westindia.json
                          kubeconfig --&gt; kubeconfig.westus2.json
                          kubeconfig --&gt; kubeconfig.westus.json
</code></pre></div></div>

<h2 id="create-a-new-resource-group-for-the-kubernetes-cluster">Create a new Resource Group for the Kubernetes Cluster</h2>

<p>Run the following command to deploy a new Resource Group.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>    az group create <span class="se">\</span>
    <span class="nt">--name</span> azure-k8s-dev <span class="se">\</span>
    <span class="nt">--location</span> westeurope
</code></pre></div></div>

<p>You should get the following response back.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">{</span>
  <span class="s2">"id"</span>: <span class="s2">"/subscriptions/d5b31b94-d91c-4ef8-b9d0-30193e6308ee/resourceGroups/azure-k8s-dev"</span>,
  <span class="s2">"location"</span>: <span class="s2">"westeurope"</span>,
  <span class="s2">"managedBy"</span>: null,
  <span class="s2">"name"</span>: <span class="s2">"azure-k8s-dev"</span>,
  <span class="s2">"properties"</span>: <span class="o">{</span>
    <span class="s2">"provisioningState"</span>: <span class="s2">"Succeeded"</span>
  <span class="o">}</span>,
  <span class="s2">"tags"</span>: null
<span class="o">}</span>
</code></pre></div></div>

<h2 id="deploy-the-kubernetes-arm-template-to-the-resource-group">Deploy the Kubernetes ARM Template to the Resource Group</h2>

<p>Run the following command to deploy the Kubernetes ARM Template to the <strong>azure-k8s-dev</strong> Resource Group.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>az group deployment create <span class="se">\</span>
    <span class="nt">--name</span> <span class="s2">"azure-k8s-dev-Deployment"</span> <span class="se">\</span>
    <span class="nt">--resource-group</span> <span class="s2">"azure-k8s-dev"</span> <span class="se">\</span>
    <span class="nt">--template-file</span> <span class="s2">"./_output/azure-k8s-dev/azuredeploy.json"</span> <span class="se">\</span>
    <span class="nt">--parameters</span> <span class="s2">"./_output/azure-k8s-dev/azuredeploy.parameters.json"</span>
</code></pre></div></div>

<p>This command should run for approximately 10 to 15 minutes. When the command completes, you should get back a very long list of output which I have ommitted from here due to its length. It’s much easier to track and verify the deployment succeeded in the <a href="https://portal.azure.com">Azure Portal</a> in the <strong>azure-k8s-dev</strong> Resource Group.</p>

<h2 id="connect-to-the-kubernetes-cluster">Connect to the Kubernetes Cluster</h2>

<p>In order to connect to the K8s Cluster, we have to point kubectl to the kubeconfig file to use. Run the following command to set the location of the kubeconfig file; because we deployed the K8s cluster in Western Europe, we are pointing to the respective kubeconfig file.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">export </span><span class="nv">KUBECONFIG</span><span class="o">=</span>~/_output/azure-k8s-dev/kubeconfig/kubeconfig.westeurope.json
</code></pre></div></div>

<p>Next, run the following command to verify that you can connect to the Kubernetes Cluster and display the clusters information.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl cluster-info
</code></pre></div></div>

<p>You should get back the following output.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>Kubernetes master is running at https://azure-k8s-dev.westeurope.cloudapp.azure.com
Heapster is running at https://azure-k8s-dev.westeurope.cloudapp.azure.com/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://azure-k8s-dev.westeurope.cloudapp.azure.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://azure-k8s-dev.westeurope.cloudapp.azure.com/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
tiller-deploy is running at https://azure-k8s-dev.westeurope.cloudapp.azure.com/api/v1/namespaces/kube-system/services/tiller-deploy:tiller/proxy
</code></pre></div></div>

<p>Next, run the following command to display the Nodes in the Cluster.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get nodes
</code></pre></div></div>

<p>You should get back the following output.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>NAME                        STATUS    ROLES     AGE       VERSION
k8s-linuxpool1-30657238-0   Ready     agent     15m       v1.9.3
k8s-linuxpool1-30657238-1   Ready     agent     18m       v1.9.3
k8s-master-30657238-0       Ready     master    18m       v1.9.3
</code></pre></div></div>

<p>Lastly, run the following command to display all of the current pods running in the cluster.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get pods <span class="nt">--all-namespaces</span>
</code></pre></div></div>

<p>You should get back the following output. The Pod names will be slightly different for you.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>NAMESPACE     NAME                                            READY     STATUS    RESTARTS   AGE
kube-system   heapster-668b9fdf67-zx7v4                       2/2       Running   0          19m
kube-system   kube-addon-manager-k8s-master-30657238-0        1/1       Running   0          19m
kube-system   kube-apiserver-k8s-master-30657238-0            1/1       Running   0          18m
kube-system   kube-controller-manager-k8s-master-30657238-0   1/1       Running   0          18m
kube-system   kube-dns-v20-55498dbf49-94hr8                   3/3       Running   0          19m
kube-system   kube-dns-v20-55498dbf49-mm8jk                   3/3       Running   0          19m
kube-system   kube-proxy-h87xw                                1/1       Running   0          19m
kube-system   kube-proxy-nhlh8                                1/1       Running   0          19m
kube-system   kube-proxy-ss9cv                                1/1       Running   0          17m
kube-system   kube-scheduler-k8s-master-30657238-0            1/1       Running   0          19m
kube-system   kubernetes-dashboard-868965c888-2jpxn           1/1       Running   0          19m
kube-system   tiller-deploy-589f6788d7-5gk95                  1/1       Running   0          19m
</code></pre></div></div>

<h2 id="closing">Closing</h2>

<p>This article covered the basics of how to quickly setup and deploy a Kubernetes Cluster running in Azure using the Azure Container Service Engine and verify it is working.</p>

  </div>

  <div class="date">
    Written on February 19, 2018
  </div>

  
<div class="comments">
	<div id="disqus_thread"></div>
	<script type="text/javascript">

	    var disqus_shortname = 'starkfell-github-io';

	    (function() {
	        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
	        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
	        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
	    })();

	</script>
	<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
</div>

</article>

    </div>

    <div class="wrapper-footer">
      <div class="container">
        <footer class="footer">
          
<a href="mailto:ryan.irujo@gmail.com"><i class="svg-icon email"></i></a>


<a href="https://github.com/starkfell"><i class="svg-icon github"></i></a>

<a href="https://www.linkedin.com/in/rirujo"><i class="svg-icon linkedin"></i></a>


<a href="https://www.twitter.com/reirujo"><i class="svg-icon twitter"></i></a>



        </footer>
      </div>
    </div>

    

  </body>
</html>
