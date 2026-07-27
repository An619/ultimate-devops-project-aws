Good. Since you already know DNS, let's focus on **what Route 53 adds**.

---

# First, what is Route 53?

**Amazon Route 53 is AWS's managed DNS service.**

Think of it like this:

```text
DNS = The concept (phonebook of the Internet)

Route 53 = AWS's implementation of DNS
```

Just like:

```text
Kubernetes = Container orchestration

EKS = AWS's implementation of managed Kubernetes
```

---

# Without Route 53

Suppose your website is running on an AWS Load Balancer.

AWS gives you:

```text
my-alb-123456.us-east-1.elb.amazonaws.com
```

Imagine asking users to visit:

```text
https://my-alb-123456.us-east-1.elb.amazonaws.com
```

Not practical.

Instead you want:

```text
https://amazon.com
```

---

# What Route 53 does

Route 53 simply maps

```text
amazon.com
```

to

```text
my-alb-123456.us-east-1.elb.amazonaws.com
```

Now the user only remembers

```text
amazon.com
```

---

# Complete Flow

Suppose you type

```text
amazon.com
```

Your browser asks

```text
What is the IP for amazon.com?
```

Route 53 replies

```text
amazon.com

↓

my-alb-123456.elb.amazonaws.com
```

Then AWS DNS resolves that further to the current IP address(es) of the load balancer.

Finally

```text
Browser

↓

AWS Load Balancer

↓

Ingress Controller

↓

Payment Service

↓

Payment Pod
```

---

# Why not directly map to an EC2 IP?

Suppose you mapped

```text
amazon.com

↓

18.220.15.10
```

Now imagine that EC2 crashes.

AWS creates another one.

New IP

```text
18.220.40.25
```

Your DNS is now wrong.

Everyone gets

```text
Server Not Found
```

---

Instead

```text
amazon.com

↓

AWS Load Balancer
```

The Load Balancer always knows which EC2 instances are healthy.

So DNS **never changes**.

---

# In Kubernetes

Now connect this to everything you've learned.

Suppose you deployed your application on EKS.

You have

```text
Pods

↓

Services

↓

Ingress

↓

AWS Application Load Balancer
```

AWS creates

```text
my-alb-123456.us-east-1.elb.amazonaws.com
```

Then Route 53 maps

```text
shop.com

↓

my-alb-123456.us-east-1.elb.amazonaws.com
```

Now when someone types

```text
shop.com/payment
```

the journey is

```text
Browser
      │
      ▼
Route 53
      │
(shop.com → ALB)
      ▼
AWS Application Load Balancer
      │
      ▼
Ingress Controller
      │
      ▼
Ingress Rules
      │
      ▼
Payment Service
      │
      ▼
Payment Pod
```

---

# Why is it called Route **53**?

This is a common interview question.

DNS servers communicate over **port 53**.

Both:

* UDP 53 (most DNS queries)
* TCP 53 (large responses, zone transfers)

AWS named the service **Route 53** after the DNS port.

---

# Besides DNS mapping, what else can Route 53 do?

This is where it becomes more powerful than a basic DNS server.

Suppose your application is deployed in two AWS regions.

```text
India

↓

ALB (Mumbai)
```

```text
Germany

↓

ALB (Frankfurt)
```

Route 53 can intelligently decide where to send users.

Examples:

### 1. Latency-based routing

```text
Indian User

↓

Mumbai ALB
```

```text
German User

↓

Frankfurt ALB
```

---

### 2. Health checks

Suppose the Mumbai ALB goes down.

Normally users would get errors.

Route 53 continuously checks health.

If Mumbai fails:

```text
Indian User

↓

Frankfurt ALB
```

Automatically.

---

### 3. Weighted routing

Suppose you're testing a new version.

```text
90% Traffic

↓

Version 1
```

```text
10% Traffic

↓

Version 2
```

Very useful for canary deployments.

---

# Mental Picture

Think of the entire Internet like this:

```text
You want to visit "Amazon"

        │
        ▼
Ask Route 53
("Where is amazon.com?")
        │
        ▼
Route 53 replies
("Go to this AWS Load Balancer")
        │
        ▼
AWS Load Balancer
("I'll forward you into the Kubernetes cluster")
        │
        ▼
Ingress Controller
("Which microservice do you need?")
        │
        ▼
Payment Service
("I'll choose a healthy Pod")
        │
        ▼
Payment Pod
("Here's your response")
```

---

## The role of Route 53 in one sentence

**Route 53 does not know anything about Pods, Services, or Kubernetes. Its only job is to answer the question: "When someone types this domain name, where should they go?"**

Once it points the request to your AWS Load Balancer, Kubernetes takes over. That's why in an EKS architecture:

* **Route 53** gets the request to your cluster.
* **AWS Load Balancer** receives the traffic.
* **Ingress Controller** decides which Service should handle it.
* **Service** chooses a healthy Pod.
* **Pod** runs your application.
