<h1>🧪 Hands-On: Agent-Based Vulnerability Scanning with Tenable Nessus Agent</h1>

<p>
In this lab, I deployed a <strong>Tenable Nessus Agent</strong> on a Windows virtual machine and executed a <strong>triggered, agent-based vulnerability scan</strong>. This walkthrough documents the full process, from VM provisioning through agent cleanup.
</p>

<hr />

<h2>☁️ Step 1: Provision a Secure Windows Virtual Machine</h2>

<p>
I provisioned a Windows virtual machine and ensured strong security hygiene during setup.
</p>

<ul>
  <li>Created a Windows virtual machine</li>
  <li>Used a strong, non-default username and password</li>
  <li>Noted the VM name to avoid deleting the wrong resource later</li>
</ul>

<p>
Avoiding default credentials is critical, as exposed VMs with weak passwords are commonly breached.
</p>

<img width="715" height="521" alt="image" src="https://github.com/user-attachments/assets/372dd000-d61d-41b7-8c9a-fb94e40bd627" />


<hr />

<h2>👥 Step 2: Create a Nessus Agent Group</h2>

<p>
To organize and manage agents, I created a dedicated agent group in Tenable.
</p>

<ul>
  <li>Navigated to <code>Settings → Sensors → Nessus Agents → Agent Groups</code></li>
  <li>Created a new agent group with a custom name</li>
</ul>

<img width="727" height="285" alt="image" src="https://github.com/user-attachments/assets/b7b9dd51-eb3e-41e4-add5-581d0d8ae304" />


<hr />

<h2>🧩 Step 3: Create a Basic Agent Scan</h2>

<p>
I created a scan designed specifically for agent-based execution.
</p>

<ul>
  <li>Created a new <strong>Basic Agent Scan</strong></li>
  <li>Selected the previously created agent group as the scan target</li>
</ul>

<img width="1397" height="699" alt="image" src="https://github.com/user-attachments/assets/d350b40b-0b90-419c-8e50-cf78a2e05345" />

<hr />

<h2>⏯️ Step 4: Configure a Triggered Scan</h2>

<p>
To control when the scan runs, I configured it as a triggered scan.
</p>

<ul>
  <li>Enabled triggered scanning</li>
  <li>Defined a trigger file (e.g., <code>start.txt</code>)</li>
  <li>Noted the trigger filename for later use</li>
</ul>

<img width="1389" height="235" alt="image" src="https://github.com/user-attachments/assets/4e555d31-9c98-458f-96ab-09f2f542f6a2" />


<hr />

<h2>🔐 Step 5: Log Into the Windows VM</h2>

<p>
I logged into the Windows virtual machine using the secure credentials created earlier.
</p>

<img width="341" height="81" alt="image" src="https://github.com/user-attachments/assets/dbaadda4-e14c-4a1b-b112-fa2d5a29b8d0" />

<hr />

<h2>📥 Step 6: Provision the Nessus Agent</h2>

<p>
From the Tenable Portal, I began provisioning a Nessus Agent.
</p>

<ul>
  <li>Navigated to <code>Settings → Sensors → Nessus Agents → Add Nessus Agent</code></li>
  <li>Copied the PowerShell installation command provided by Tenable</li>
</ul>

<img width="353" height="429" alt="image" src="https://github.com/user-attachments/assets/77384700-e06a-4f3b-9244-d298da6f90a4" />

<hr />

<h2>📝 Step 7: Customize the PowerShell Install Command</h2>

<p>
Before execution, I edited the PowerShell command to match my environment.
</p>

<ul>
  <li>Pasted the command into Notepad</li>
  <li>Updated the agent name and group values</li>
  <li>Verified the command syntax</li>
</ul>

<img width="1099" height="231" alt="image" src="https://github.com/user-attachments/assets/080c2f6b-ccb8-40db-96a7-664c9ae28bb2" />


<hr />

<h2>⚡ Step 8: Install the Agent on the VM</h2>

<p>
I installed the agent directly on the VM.
</p>

<ul>
  <li>Opened PowerShell as Administrator</li>
  <li>Pasted and executed the installation command</li>
  <li>Verified the installation completed successfully</li>
</ul>

<p>
If errors occurred, I reviewed the output and confirmed the provisioning steps and command syntax.
</p>

<img width="649" height="67" alt="image" src="https://github.com/user-attachments/assets/7f540ae9-60ba-43c2-accc-851fe81bb87b" />

<img width="623" height="171" alt="image" src="https://github.com/user-attachments/assets/6b1900d8-de75-4245-a15b-a54a39e25b89" />


<hr />

<h2>📂 Step 9: Trigger the Local Agent Scan</h2>

<p>
To initiate the scan, I created the trigger file on the VM.
</p>

<ul>
  <li>Navigated to the trigger directory</li>
  <li>Created the trigger file (e.g., <code>start.txt</code>)</li>
  <li>Watched for the file to disappear, indicating the scan had started</li>
</ul>

<img width="693" height="217" alt="image" src="https://github.com/user-attachments/assets/fba37803-d4da-48ae-8517-c07bba5d9356" />

<hr />

<h2>🛰️ Step 10: Verify Agent Linkage in Tenable</h2>

<p>
I confirmed the agent successfully linked to Tenable Cloud.
</p>

<ul>
  <li>Returned to <code>Settings → Sensors → Nessus Agents</code></li>
  <li>Verified the agent appeared with a recent “Linked On” timestamp</li>
  <li>Confirmed the agent name matched my VM</li>
</ul>

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/708e8a8b-bfa6-422e-bd37-7db6d352e9f4" />


<hr />

<h2>⏳ Step 11: Monitor Scan Progress</h2>

<p>
Agent-based scans require time to populate findings.
</p>

<ul>
  <li>Waited 30–60 minutes for vulnerabilities to populate</li>
  <li>Checked <code>Scans → See All Details</code></li>
</ul>

<img width="2501" height="479" alt="image" src="https://github.com/user-attachments/assets/4b4cae8a-699f-4618-b08b-6f4378e9cdaf" />

<hr />

<h2>📊 Step 12: Review Vulnerability Results</h2>

<p>
Once complete, I reviewed the findings.
</p>

<ul>
  <li>Observed detected vulnerabilities</li>
  <li>Reviewed severity ratings and affected components</li>
  <li>Validated the effectiveness of local agent scanning</li>
</ul>

<img width="1925" height="415" alt="image" src="https://github.com/user-attachments/assets/530fea3f-820f-4b9e-a03b-806e2813faf0" />


<hr />

<h2>🧹 Step 13: Cleanup & Deprovisioning</h2>

<p>
After analysis, I removed all testing artifacts.
</p>

<ul>
  <li>Deleted the scan</li>
  <li>Deleted the agent group</li>
  <li>Unlinked the Nessus Agent</li>
  <li>Deleted the Windows virtual machine</li>
</ul>

<p>
This ensures security, cost control, and proper lab hygiene.
</p>


<hr />

<h2>🎯 Skills Demonstrated</h2>

<ul>
  <li>Agent-based vulnerability scanning</li>
  <li>Nessus Agent deployment and management</li>
  <li>PowerShell execution</li>
  <li>Endpoint security visibility</li>
  <li>SOC & Vulnerability Management operations</li>
</ul>
