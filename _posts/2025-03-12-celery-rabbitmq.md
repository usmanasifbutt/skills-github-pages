---
title: "Using Celery with RabbitMQ: Installation & Queue Setup"
date: 2025-03-12
author: "Usman Asif"
description: "A step-by-step guide on setting up Celery with RabbitMQ, including installation, queue creation, and task execution."
---


## 🚀 Introduction

Celery is a powerful distributed task queue system, and RabbitMQ is one of its most commonly used brokers. This guide will walk you through setting up Celery with RabbitMQ, configuring queues, and running tasks efficiently.

----------


## 📌 Installation

### 1️⃣ Install Docker

To run RabbitMQ, you need Docker installed on your system. If you don’t have it, install it using:

{% highlight python %}
brew install docker  # macOS (use apt or yum for Linux)
{% endhighlight %}

### 2️⃣ Start RabbitMQ

You can start RabbitMQ using a simple Docker command. First, create an alias for convenience:

{% highlight python %}
alias rmq="docker run -d --name rabbitmq-3 -p 5672:5672 -p 15672:15672 rabbitmq:3-management"
{% endhighlight %}

Now start RabbitMQ with:

{% highlight python %}
rmq
{% endhighlight %}

### 3️⃣ Verify the Installation

Check if RabbitMQ is running properly by querying the queues:

{% highlight python %}
curl 'http://guest:guest@localhost:15672/api/queues'
{% endhighlight %}

If RabbitMQ is running, it should return `[]` (an empty list of queues). If there are errors, run RabbitMQ without `-d` to see logs:

{% highlight python %}
docker run --name rabbitmq-3 -p 5672:5672 -p 15672:15672 rabbitmq:3-management
{% endhighlight %}

----------

## 📌 Setting Up Celery

### 1️⃣ Install Celery

If you haven't installed Celery, do so using pip:

{% highlight python %}
pip install celery
{% endhighlight %}

### 2️⃣ Configure Celery with RabbitMQ

Create a `celery.py` file in your Django or Python project:

{% highlight python %}
from celery import Celery

app = Celery('my_project', broker='pyamqp://guest@localhost//')

@app.task
def add(x, y):
    return x + y
{% endhighlight %}

### 3️⃣ Running Celery Worker

Start a Celery worker with:

{% highlight python %}
celery -A celery worker --loglevel=info
{% endhighlight %}

You should see output indicating Celery is connected to RabbitMQ.

----------

## 📌 Creating & Managing Queues

RabbitMQ allows you to define and manage queues explicitly. You can do this via the management UI (`http://localhost:15672`) or programmatically:

### 1️⃣ Define a Queue in Celery

Modify `celery.py` to route tasks to specific queues:

{% highlight python %}
app.conf.task_routes = {
    'tasks.add': {'queue': 'math_queue'},
}
{% endhighlight %}

Then, start a worker for that queue:

{% highlight python %}
celery -A celery worker -Q math_queue --loglevel=info
{% endhighlight %}

### 2️⃣ Creating a Dead Letter Queue (DLQ)

To handle failed tasks, configure a DLQ:

{% highlight python %}
app.conf.task_queues = {
    Queue('math_queue', exchange=Exchange('math', type='direct'), routing_key='math'),
    Queue('dlq', exchange=Exchange('dlx', type='direct'), routing_key='dlq'),
}
{% endhighlight %}

This ensures failed tasks are sent to `dlq` for later processing.

----------

## 📌 Executing Tasks

To test your Celery setup, open a Python shell:

{% highlight python %}
from celery import Celery
app = Celery('my_project', broker='pyamqp://guest@localhost//')

result = app.send_task('tasks.add', args=[10, 5], queue='math_queue')
print(result.get())
{% endhighlight %}

This should output `15` after processing the task.

----------

## 🎯 Conclusion

You’ve now set up Celery with RabbitMQ, created custom queues, and executed tasks asynchronously. This setup ensures scalable and reliable background task execution.

Happy coding! 🚀