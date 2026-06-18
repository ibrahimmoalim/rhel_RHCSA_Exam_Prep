The `rhc connect` utility (Remote Host Configuration) relies on **Activation Keys** and an **Organization ID** rather than your typical developer username and password.

Since you already have a Red Hat Individual Developer account, you can easily generate these credentials to get Red Hat Lightspeed and Insights hooked up.

---

### Get your Organization ID and Activation Key

1. Open your browser and log into the **[Red Hat Hybrid Cloud Console](https://www.google.com/search?q=https://console.redhat.com/)** using your developer account credentials.
2. In the left navigation sidebar, go to **Services** -> **System Configuration** -> **Activation Keys**.
3. At the top of the *Activation Keys* page, you will see your numeric **Organization ID** (e.g., `12345678`). Write this down.
4. Click the blue **Create activation key** button:
* **Name:** Give it a memorable name (like `dev-sandbox`).
* **Workload:** Choose *Latest release*.
* **System Purpose:** (Optional) You can leave this blank or select *Development/Test*.


5. Click **Create**.

---

### Run the Connect Command on your Server

Switch back to your server's terminal. Run the connection command as root or via `sudo`, substituting your Organization ID and the Key Name you just created:

```bash
sudo rhc connect --organization <YOUR_ORG_ID> --activation-key <YOUR_KEY_NAME>

```

---

### Install the Execution Worker

To completely enable Lightspeed's remote execution features, smart remediation, and playbook automation, install the terminal worker package right after connecting:

```bash
sudo dnf install -y rhc-worker-playbook

```

### Verify the Connection

Run the status check tool to make sure everything went through cleanly:

```bash
sudo rhc status

```

You should see checkmarks ($\checkmark$) confirming that you are successfully connected to both the Red Hat Subscription Manager and Red Hat Lightspeed/Insights.
