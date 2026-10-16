# Cloud Computing (DS 584) - Assignment 01: Hosting & Load Balancing a Web Application

**Faculty of Computing and Information Technology (FCIT), University of the Punjab**  
**Course:** DS 584 - Cloud Computing (Fall 2026)  
**Instructor:** Dr. Muhammad Abdullah  
**Student:** Muhammad Imran Mian  
**Roll No:** MSDSF25-A007  

---

## 📌 Project Overview

This repository contains the complete implementation, architecture design, and performance benchmarking for **Assignment 01: Hosting and Load Balancing a Web Application Using Ubuntu Virtual Machines**.

The project demonstrates:
1. Building a web application (Quadratic Equation Solver) in **Python Flask** with a responsive **Bootstrap 5** and **jQuery AJAX** frontend.
2. Generating random quadratic equations on the server, solving for real/complex roots, and returning host diagnostic metadata (hostname and server IP).
3. Deploying the application redundantly across two standalone **Ubuntu Server Virtual Machines** (VM1 & VM2).
4. Configuring a third Ubuntu VM as an **Nginx Reverse Proxy & Layer 7 Load Balancer** with round-robin traffic distribution.
5. Executing workload stress testing and performance analysis using **Postman** / **Apache JMeter** and system resource profiling.

---

## 🏗️ Architecture Diagram

```
                              Client Web Browser / Stress Tester
                                              │
                                              ▼
                             ┌─────────────────────────────────┐
                             │    Nginx Reverse Proxy & LB     │
                             │        (Ubuntu VM 3)            │
                             │     192.168.100.158:5000        │
                             └────────────────┬────────────────┘
                                              │
                       ┌──────────────────────┴──────────────────────┐
            1:1 Round Robin Traffic                       1:1 Round Robin Traffic
                       │                                             │
                       ▼                                             ▼
        ┌─────────────────────────────┐               ┌─────────────────────────────┐
        │   Backend App Server 1      │               │   Backend App Server 2      │
        │       (Ubuntu VM 1)         │               │       (Ubuntu VM 2)         │
        │ 192.168.100.156 / .161:5000 │               │    192.168.100.157:5000     │
        │       [Flask + WSGI]        │               │       [Flask + WSGI]        │
        └─────────────────────────────┘               └─────────────────────────────┘
```

---

## 🚀 Milestones Breakdown & Solution Techniques

### Milestone 01: Quadratic Equation Solver Web Application
- **Backend (`app.py`)**:
  - Developed in **Python** using the **Flask** microframework.
  - Generates random integer coefficients $a, b, c \in [-10, 10]$ ensuring $a \neq 0$.
  - Computes discriminant $D = b^2 - 4ac$ and handles:
    - **Two Real Roots** ($D > 0$): $x = \frac{-b \pm \sqrt{D}}{2a}$
    - **One Real Root** ($D = 0$): $x = \frac{-b}{2a}$
    - **Complex Roots** ($D < 0$): Real and imaginary components rendered as $u \pm vi$
  - Dynamically discovers the active network interface IP and machine hostname using Python's `socket` library.
  - Exposes `/solve` JSON API endpoint.
- **Frontend (`templates/index.html`)**:
  - Responsive UI styled with **Bootstrap 5**.
  - **jQuery AJAX** asynchronous calls to `/solve` on initial load and upon clicking **"Generate & Solve Equation"**.
  - Displays formatted equation string, coefficients, root type, calculated solutions, and live **Server Diagnostics** (hostname + server IP address).

### Milestone 02: Infrastructure Provisioning & Deployment on Ubuntu VMs
- Provisioned two **Ubuntu Server Virtual Machines** using hypervisor virtualization (Virtual Machine Manager / VirtualBox).
- **Network Configuration**:
  - Configured with a **Bridged Network Adapter** so each VM receives an IP address within the host subnet (`192.168.100.x`), directly accessible from the host OS browser.
- **Software Stack**:
  - Python 3, `pip`, `virtualenv`, Flask, and Gunicorn (`requirements.txt`).
- **Application Execution**:
  - Backend runs on `0.0.0.0:5000` on both VM1 and VM2.
  - Verified independent accessibility via:
    - **VM 1:** `http://192.168.100.156:5000/` (or `http://192.168.100.161:5000/`)
    - **VM 2:** `http://192.168.100.157:5000/`

### Milestone 03: Nginx Proxy Server & Load Balancer Setup
- Provisioned a dedicated 3rd Ubuntu VM for **Nginx**.
- Installed Nginx via standard package manager:
  ```bash
  sudo apt update && sudo apt install -y nginx
  ```
- Configured upstream backend cluster and reverse proxy routing (`/etc/nginx/sites-available/default`):
  ```nginx
  upstream flask_backend {
      server 192.168.100.161:5000;
      server 192.168.100.157:5000;
  }

  server {
      listen 5000;

      location / {
          proxy_pass http://flask_backend;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }
  ```
- **Load Balancing Verification**:
  - Loaded `http://192.168.100.158:5000/` through host browser.
  - Repeated clicks alternated between **VM1** and **VM2** hostnames and IP addresses in the **Server Diagnostics** panel, confirming smooth 1:1 round-robin load distribution.

### Milestone 04: Synthetic Workload Generation & Performance Profiling
- Evaluated performance, response times, and server stability under synthetic concurrency workloads.
- **Stress Testing Tools**:
  - **Postman Runner / Stress Testing**: Conducted load tests comparing direct VM endpoints vs. the Nginx load-balanced endpoint.
  - **Apache JMeter & JMeter Plugins**: Ultimate Thread Group (`jp@gc - Ultimate Thread Group.jmx`) ramp-up concurrency testing from 10 to 1000 users.
  - **ServerAgent Resource Profiler**: Monitored CPU utilization and response latencies across backends.
- **Observations**:
  - Nginx efficiently distributed concurrent client requests across both backend VMs.
  - Balanced throughput prevented single-point server degradation, maintaining consistent response times across test runs.

---

## 📂 Repository Structure

```
.
├── app.py                     # Flask application logic, solver, and server diagnostics
├── requirements.txt           # Python dependencies (Flask, Gunicorn)
├── templates/
│   └── index.html             # Bootstrap 5 + jQuery AJAX user interface
├── response time logs/
│   ├── jp@gc - Ultimate Thread Group.jmx   # JMeter load testing thread group configuration
│   └── Response Time Graph.png             # Performance analysis and latency graph
├── CC_Assignment1_MSDSF25A007.pdf          # Final comprehensive project report
├── Assignment_1__Hosting_and_Load_Balancing_a_Web_Application_Fall_2026.pdf # Problem specification
└── README.md                  # Project documentation
```

---

## 🛠️ How to Run Locally

### 1. Clone the Repository
```bash
git clone https://github.com/memrranmian/cloud-computing-load-balanced-webapp.git
cd cloud-computing-load-balanced-webapp
```

### 2. Set Up Virtual Environment & Dependencies
```bash
python -m venv venv
# On Windows:
venv\Scripts\activate
# On Linux/macOS:
source venv/bin/activate

pip install -r requirements.txt
```

### 3. Run the Flask Web Application
```bash
python app.py
```
Open your browser and navigate to `http://127.0.0.1:5000/`.

---

## 🌐 Deploying with Nginx on Linux

1. Run the Flask app with Gunicorn on both backend VMs:
   ```bash
   gunicorn -w 3 -b 0.0.0.0:5000 app:app
   ```
2. On the Proxy VM, edit `/etc/nginx/sites-available/default`, add the `upstream` block with the IP addresses of your backend VMs, and reload Nginx:
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```
3. Access the proxy IP at port `5000` to verify round-robin routing.

---

## 👤 Author
- **Muhammad Imran Mian** (Roll No: MSDSF25-A007)
- Master of Science in Data Science (MSDS)
- Faculty of Computing and Information Technology (FCIT), University of the Punjab
