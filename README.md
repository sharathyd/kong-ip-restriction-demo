# Kong IP Restriction Plugin Demo

This repository demonstrates use of Kong's IP restriction plugin to deny or pass through the incoming request based on IP address.
It has step-by-step commands and outputs for setting up a **Kong service**, **route**, and applying the **IP Restriction plugin** in a workspace (`demo`).

---

## Table of Contents
1. [Create a Service](#create-a-service)
2. [Create a Route](#create-a-route)
3. [Add IP Restriction Plugin](#add-ip-restriction-plugin)
4. [Configure Trusted IPs](#configure-trusted-ips)
5. [Test API with Allowed IP](#test-api-with-allowed-ip)
6. [Test API with Blocked IP](#test-api-with-blocked-ip)
7. [Modify Plugin to Deny List](#modify-plugin-to-deny-list)
8. [Test Deny List - Allowed IP](#test-deny-list---allowed-ip)
9. [Test Deny List - Blocked IP](#test-deny-list---blocked-ip)

---

## Create a Service

```bash
curl -i -X POST http://localhost:8001/demo/services --data "name=httpbin_demo_svc" --data "url=https://httpbin.konghq.com/anything"
```

**Output:**
```
HTTP/1.1 201 Created
{"id":"700addd1-efd3-42ff-9989-019ede807914","name":"httpbin_demo_svc","host":"httpbin.konghq.com","path":"/anything","protocol":"https"}
```

---

## Create a Route

```bash
curl -i -X POST http://localhost:8001/demo/services/httpbin_demo_svc/routes   --data "name=httbin_demo_rt"   --data "paths[]=/demo-anything"
```

**Output:**
```
HTTP/1.1 201 Created
{"id":"7fb4e4f4-9143-4aaa-80fd-be8bd18bfa9c","name":"httbin_demo_rt","paths":["/demo-anything"]}
```

---

## Add IP Restriction Plugin

```bash
curl -i -X POST http://localhost:8001/demo/services/httpbin_demo_svc/plugins   --data "name=ip-restriction"   --data "config.allow=127.0.0.1"   --data "config.status=403"   --data "config.message=Your IP address is not allowed"
```

**Output:**
```
HTTP/1.1 201 Created
{"id":"f8364a15-b950-449f-bf47-5fb421408137","name":"ip-restriction","config":{"allow":["127.0.0.1"],"status":403,"message":"Your IP address is not allowed"}}
```

---

## Configure Trusted IPs

Set the following environment variables in **Kong Data Plane** and restart Kong:

```bash
export KONG_TRUSTED_IPS=0.0.0.0/0
export KONG_REAL_IP_HEADER=X-Forwarded-For
```

---

## Test API with Allowed IP

```bash
curl -i --request GET --url http://localhost:8000/demo-anything --header 'X-Forwarded-For:127.0.0.1'
```

**Output:**
```
HTTP/1.1 200 OK
{"url": "http://localhost/anything", "origin": "127.0.0.1"}
```

---

## Test API with Blocked IP

```bash
curl -i --request GET --url http://localhost:8000/demo-anything --header 'X-Forwarded-For:192.168.0.1'
```

**Output:**
```
HTTP/1.1 403 Forbidden
{"message":"Your IP address is not allowed"}
```

---

## Modify Plugin to Deny List

```bash
curl -i -X POST http://localhost:8001/demo/services/httpbin_demo_svc/plugins   --data "name=ip-restriction"   --data "config.allow="   --data "config.deny=200.0.0.0/24"   --data "config.status=403"   --data "config.message=Blocked by denylist"
```

---

## Test Deny List - Allowed IP

```bash
curl --request GET --url http://localhost:8000/demo-anything --header 'X-Forwarded-For:127.0.0.1'
```

**Output:**
```
HTTP/1.1 200 OK
{"url": "http://localhost/anything", "origin": "127.0.0.1"}
```

---

## Test Deny List - Blocked IP

```bash
curl --request GET --url http://localhost:8000/demo-anything --header 'X-Forwarded-For:200.0.0.1'
```

**Output:**
```
HTTP/1.1 403 Forbidden
{"message":"Blocked by denylist"}
```

