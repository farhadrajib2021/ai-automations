🍽️ AI Kitchen Printing System – Full Setup Guide

(Reusable for Any Restaurant)

This guide explains how to set up an AI-powered kitchen printing system that allows orders to be printed automatically without waiters running back and forth.

Orders can come from:

QR code menu

WhatsApp

Website

Tablet

AI Assistant

And are printed instantly in the kitchen.

🧩 System Overview (Simple)
Customer → AI Order System → Print API → POS Printer


No waiter needed to carry orders

No tablet in the kitchen

No cloud printer subscription

Works 24/7

🧰 What You Need
Hardware

✅ Raspberry Pi (any model: 3 / 4 / 5)

✅ Thermal POS Printer (USB or LAN)

✅ Wi-Fi connection

✅ Thermal paper (80mm recommended)

Software

Linux (Raspberry Pi OS)

CUPS (printing system)

Node.js

n8n (or any automation platform)

Cloudflare Tunnel (secure internet access)

1️⃣ Connect the Printer to Raspberry Pi

Plug the printer into the Raspberry Pi (USB or LAN)

Power on the printer

Install printing system:

sudo apt update
sudo apt install cups


Enable & start CUPS:

sudo systemctl enable cups
sudo systemctl start cups


Add the printer:

Open browser on Pi → http://localhost:631

Add printer

Choose Generic Text-Only Printer

Name example: kitchen_printer

Test:

echo "TEST PRINT" | lp -d kitchen_printer


✅ If it prints → printer setup is complete

2️⃣ Install the Print API (Node.js)
Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo bash
sudo apt install -y nodejs

Create project
mkdir print-api
cd print-api
npm init -y
npm install express body-parser

3️⃣ Print API (server.js)

This server receives plain text and prints it.

const express = require("express");
const bodyParser = require("body-parser");
const { exec } = require("child_process");

const app = express();
app.use(bodyParser.text({ type: "*/*" }));

const PRINTER_NAME = "kitchen_printer";

app.post("/print", (req, res) => {
  let content = req.body;

  if (!content || content.trim() === "") {
    return res.status(400).send("Empty print body");
  }

  // Add space top & bottom + auto cut
  const ESC_CUT = "\x1D\x56\x00";      // Full cut
  const FEED_TOP = "\n\n\n";
  const FEED_BOTTOM = "\n\n\n\n";

  const finalText = FEED_TOP + content + FEED_BOTTOM + ESC_CUT;
  const safe = finalText.replace(/"/g, '\\"');

  exec(`printf "%b" "${safe}" | lp -d ${PRINTER_NAME}`, (err) => {
    if (err) {
      return res.status(500).send("Print failed");
    }
    res.send("Printed");
  });
});

app.listen(3000, () => {
  console.log("🔥 Print API running on port 3000");
});

Start server
node server.js

4️⃣ Auto-Start on Boot (No SSH Needed)

Create service:

sudo nano /etc/systemd/system/print-api.service


Paste:

[Unit]
Description=Print API
After=network.target cups.service

[Service]
ExecStart=/usr/bin/node /home/ricenspice/print-api/server.js
WorkingDirectory=/home/ricenspice/print-api
Restart=always
User=ricenspice
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target


Enable:

sudo systemctl daemon-reload
sudo systemctl enable print-api
sudo systemctl start print-api


✅ Printer now works after reboot automatically

5️⃣ Expose Secure Internet URL (Cloudflare)

Why?

No port forwarding

No public IP

Secure HTTPS

Install Cloudflare Tunnel
sudo apt install cloudflared
cloudflared tunnel login

Create tunnel
cloudflared tunnel create printserver

Config
sudo nano /etc/cloudflared/config.yml

tunnel: printserver
credentials-file: /home/ricenspice/.cloudflared/XXXX.json

ingress:
  - hostname: printer.restaurant-domain.com
    service: http://localhost:3000
  - service: http_status:404


Run:

cloudflared tunnel run printserver

6️⃣ Connect n8n (or Any System)
HTTP Request Node

Method: POST

URL:

https://printer.restaurant-domain.com/print


Headers:

Content-Type: text/plain


Body (Raw Text):

NEUE BESTELLUNG
------------------------------

Name: {{ customerName }}
Tisch: {{ tableNumber }}

Bestellung:
{{ items }}

Zeit: {{ timestamp }}

------------------------------
Restaurant AI


✅ Prints instantly in kitchen

7️⃣ What the Restaurant Gets
✅ Benefits

No waiter needed to carry orders

Faster service

No tablet in kitchen

No printer subscriptions

Works offline inside restaurant

Secure remote access

Scales to multiple printers

✅ Use Cases

Buffet orders

Drinks only

Kitchen tickets

Bar tickets

Delivery slips
