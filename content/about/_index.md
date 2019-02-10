+++
date = "2015-06-20T14:02:37+02:00"
title = "About"
hidden = true
+++
  
I am a software engineer who enjoys coding. I am specially interested in parallel computing, distributed systems and functional programming. 

The main aim on this website is keeping record of some of the issues that I have faced at work, so this could help to solve out similar problems on the future. By the way, much better if someone finds this website helpful
***

## Main Skills
<pre>

</pre>

* **Programming languagues**:  *Elixir, Python, Javascript, Bash, Go, Javascript ES6, Elm, C*. Although my prefer languague is Elixir (developing OTP functionalities), I have worked in other languagues like Python or Javascript. 

* **Devops**: *Linux, Docker, Nginx, Kubernetes.* Every application I have worked on is designed to be dockerized and to be launched in a cloud provider like Azure or OVH over Kubernetes.

* **Databases**: *MongoDB, Postgresql and Mysql*.

* **Networking**: *HTTP, IP, TCP, DNS, NAT*. My MSc in Telecommunication Engineering gives me a solid background about networking and OSI layers.  


<pre>

</pre>

## Experience:

<pre></pre>

<pre></pre>

### [03/2017-Currently] Software Developer at Palmtree Statistics
As a member of a little team of developers on charge of designing, developing and supporting differents applications. I have been involved in the full cycle of development of our products.

I have been developing from scratch projects and adding features in others mainly with Elixir using OTP patterns, ETS cache, Phoenix framework with Channels and Ecto. Some of main libraries I have used are Cowboy, Poolboy and Distillery.

The environment consists in a microservices architecture with more than 20 app containerized in Dockers orchestrated by Kubernetes.

Some of the main project I have worked on:

* **Analysis Project**. This applicaction launches some Python analysis from Elixir when a trigger event happens. It requires to collect data from a RabbitMQ broadcast and save the data on ETS until the events happen. Using MongoDB to store the results. It includes the development of a simple client with Phoenix to expose the results.  

* **Like/Dislike System**. Developing the cliente side on JS and the backend side with phoenix and Elixir. Real time updates using channels of Phoenix. This functionality includes realtime statistics to evaluate the opinion per each day. 

* **Analitics Service**. Connecting to an external Api Rest to request some data analysis.

* **Database Data Exporter**. Tool written in Python in charge of daily exporting and cleaning data from some internal database to an outside database, for later analysis. It allows to export data from Postgresql and MongoDB. There is up to 10 GB of data moved each day.

<pre></pre>

### [09/2016-03/2017] Intern at Palmtree Statistics   
At my internship at Palmtree I learnt mainly about Python and Docker development. I worked on the building of a testing sevice which consists in a MySQL database, an API Rest service, built with Flask in Python, where each app was contanarized in different Dockers.  

&nbsp;
## Education:
<pre></pre>

### [09/2015-09/2017] MSc in Telecomunication Engineering  (2015 - 2017). University of Malaga (Spain).

As Master thesis I implemented a Convolution Neural Network
based on VAE (Variational Autoencoder) for MRI and PET images reconstruction. It was
developed in Python and TensorFlow.

The code is available at this repository https://github.com/jkmrto/VAE-applied-to-Nifti-imagesC

### [09/2011-09/2015] BSc in Telecommunications Engineering  (2011 - 2015). University of Granada (Spain).

As Degree thesis I implemented a clustering algorithm in charge of identifying network traffic based on his package size. This project was developed in C.  

The code is available at this repository https://github.com/jkmrto/MSBC