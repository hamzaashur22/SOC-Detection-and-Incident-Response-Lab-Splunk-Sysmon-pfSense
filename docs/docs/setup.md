Alert Webhook Setup

Goal: allow the Splunk VM to remotely run an isolation command on the pfSense firewall.

Enable SSH key-based access (Splunk VM → pfSense)

Generate a public/private key pair on the Splunk VM. 

<img width="1196" height="693" alt="image" src="https://github.com/user-attachments/assets/ae13ac78-3597-426c-92bc-68e3bc5bbe65" /> 

Copy the public key.

Paste the public key into pfSense’s authorized_keys via the pfSense web UI.

<img width="1011" height="736" alt="image" src="https://github.com/user-attachments/assets/beb13189-aa9b-42f1-a4ba-1248ff267c19" />

Verify SSH key auth works

Confirm you can SSH from the Splunk VM to pfSense without being prompted for a password.

<img width="948" height="660" alt="image" src="https://github.com/user-attachments/assets/b3c1f702-9856-40af-be3f-e29c7da3e299" /> 

Test the isolation command manually first

Run the isolation command directly to confirm it works before automating it.

<img width="1081" height="505" alt="image" src="https://github.com/user-attachments/assets/0005a78f-1840-47b2-9de6-1897577dd014" />

Create the script Splunk will execute

Save the script in a directory Splunk can access.
path I used:
C:\Program Files\Splunk\bin\scripts

Ensure the Splunk service account can read/execute the script.
Note: for this lab I granted broad permissions in a real environment this would usually be restricted.

<img width="1027" height="668" alt="image" src="https://github.com/user-attachments/assets/5ccf4d8a-e314-4e4a-b51f-7b6eedca528a" /> 

Configure the Splunk alert action

In the alert settings, configure the alert action to run the script(splunk has a webhook section where you can drop your file in).

<img width="804" height="659" alt="image" src="https://github.com/user-attachments/assets/a593d7bd-b5ff-48f6-b5fc-e104f2cf077c" />
