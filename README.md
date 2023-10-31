# Identify-Application-Vulnerabilities-with-Security-Command-Center
### Overview 
I will use Web Security Scanner—one of Google Cloud Security Command Center's built-in services—to scan a Python Flask application for vulnerabilities. Web Security Scanner identifies security vulnerabilities in your App Engine, Google Kubernetes Engine (GKE), and Compute Engine web applications.

### Scenario
<br/>
<img src="https://i.imgur.com/SAc5H2c.png" height="80%" width="80%" alt="cymbal bank logo"/>
<br />
Cymbal Bank is an American retail bank with over 2,000 branches in all 50 states. It offers comprehensive debit and credit services that are built on top of a robust payments platform. Cymbal Bank is a digitally transforming legacy financial services institution.
Cymbal Bank was founded in 1920 under the name Troxler. Cymbal Group acquired the company in 1975 after it had been investing heavily in Cymbal Group's proprietary ATMs. As the bank grew into a national leader, they put strategic emphasis on modernizing the customer experience both in-person at their branches and digitally through an app they released in 2014. Cymbal Bank employs 42,000 people nationwide and, in 2019, reported $24 billion in revenue.
Cymbal Bank is interested in developing a new banking application for their corporate clients using Google Cloud technology. Application security is critical, and the CTO wants to see how Google Cloud can identify and mitigate application security vulnerabilities. As a Cloud Security Engineer, I was tasked with demonstrating Security Command Center's cutting-edge application vulnerability scanning features.

### Objectives
I will perform the following tasks:
- Launch a vulnerable Python Flask application on a Compute Engine instance
- Use Web Security Scanner to scan the application and find vulnerabilities
- Fix the application vulnerability
- Scan the application again and verify vulnerabilities no longer exist

### Setup and requirements
- a new Google Cloud project

#### Task 1. Launch a Virtual machine and deploy a vulnerable application
In this task, I will set up the infrastructure to demonstrate an application vulnerability to Cymbal Bank's CTO. More specifically, I will deploy a virtual machine, obtain the application code and introduce a vulnerability that will be detected by Web Security Scanner.<br>
	1. On the Google Cloud Console title bar, click <b>Activate Cloud Shell</b><br>
	2. Create a static IP address that will be used for scanning a vulnerable web application:
```
gcloud compute addresses create xss-test-ip-address --region=us-central1<br>
```
3. Run the following command to output the static IP address you just generated:
```
gcloud compute addresses describexss-test-ip-address \
--region=us-central1 --format="value(address)"
```
4. Run the following command to create a VM instance to run the vulnerable application:
```
gcloud compute instances create xss-test-vm-instance \
--address=xss-test-ip-address --no-service-account \
--no-scopes --machine-type=e2-micro --zone=us-central1-a \
--metadata=startup-script='apt-get update; apt-getinstall -y python3-flask'
```
<b>The startup script will install python-flask, a Web Application Framework, which is used for running a simple Python application demonstrating cross-site scripting (XSS) vulnerability, which is a common web application security vulnerability.</b>
5. Open a firewall rule for Web Security Scanner to access a vulnerable application.
```
gcloud compute firewall-rules create enable-wss-scan \
--direction=INGRESS --priority=1000 \
--network=default --action=ALLOW \
--rules=tcp:8080 --source-ranges=0.0.0.0/0
```
	6. Open the navigation menu and select Compute Engine > VM Instances.
	7. Then click on the SSH button next to your instance:


This will open an SSH connection to your VM instance in a new window.
8. In this SSH window (Not in Cloud Shell), run the following command to download and extract the vulnerable web application files:
```
gsutil cp gs://cloud-training/GCPSEC-ScannerAppEngine/flask_code.tar  . && tar xvf flask_code.tar
```
9. Now run the following command to deploy your application:
```
python3 app.py
```
10. Replace YOUR_EXTERNAL_IP in the URL field below with that IP address, and open the URL in a new browser tab:
```
http://<YOUR_EXTERNAL_IP>:8080
```
Note: You can also find the external IP address in the Google Cloud Console, where it's listed as a field associated with your VM instance.
	11. A Cymbal Bank corporate banking portal with a web form should appear.
 <img src="https://i.imgur.com/0DBVnlA.png" height="80%" width="80%" alt="cymbal banking web portal"/><br />
12. In the web form enter the following string:
```
<script>alert('This is an XSS Injection')</script>
```
13. Now press the <b>POST</b> button.
You should see the following alert window:
<img src="https://i.imgur.com/am7sL0p.png" height="80%" width="80%" alt="cymbal banking alert window"/><br />
This is a common vulnerability in web applications: a cross-site scripting vulnerability. Cross-site scripting (XSS) is a vulnerability that enables attackers to run malicious scripts in users' browsers in the context of your application.

### Task 2. Scan the application with Web Security Scanner
Now that I've launched the vulnerable application, it's time to demonstrate Web Security Scanner's abilities to the CTO. In this task, I will configure and set up a scan of the application to find security vulnerabilities.<br>
	1. Switch back to the browser tab displaying the Cloud console.<br>
	2. Open the Navigation menu and select APIs & Services > Library.<br>
	3. In Search for APIs and services type Web Security Scanner.<br>
	4. Click Enable API to enable the Web Security Scanner API.<br>
	5. Open the Navigation menu and select Security > Web Security Scanner.<br>
	6. Click + New Scan.<br>
	7. The Starting URLs field should be pre-populated with your static IP address.<br>
	8. Add the port number 8080, so that the Starting URL looks like the following:<br>
```
  http://<EXTERNAL_IP>:8080
```
9. Take a minute to review the remaining fields on the Create a new scan screen:<br>
- Authentication: a property that can be used to provide application credentials to allow the scanner to authenticate to an app while scanning.
- Schedule: a property that can be used to schedule scans to run automatically.
- Export to Security Command Center: a property that allows you to automatically export scan configurations and scan results to Cloud Security Command Center after scans are finished.
10. Verify the Authentication is still set to None and that Schedule is set to Never.<br>
11. Click Save to create the scan.<br>
Note: This creates the scan, but do not run it yet.<br>
12. Click Run to start the scan.<br>
Note: Given the number of possible tests, this can take a little over 10 minutes to scan.
13. Return to your SSH session in your separate window.<br>
If the session timed out, run the following command to restart your application:
```
python3 app.py
```
In the SSH Window, logs will begin to be generated.
<img src="https://i.imgur.com/WBgfp3B.png" height="80%" width="80%" alt="ssh logs"/><br />
14. When the scan is done running, the Results tab should indicate the cross-site vulnerabilities.<br>
<img src="https://i.imgur.com/aRLp23g.png" height="80%" width="80%" alt="cymbal banking scan result1"/><br />
<img src="https://i.imgur.com/wyis5ZX.png" height="80%" width="80%" alt="cymbal banking scan result1"/><br />
<img src="https://i.imgur.com/7Ae5Fy4.png" height="80%" width="80%" alt="cymbal banking scan result1"/><br />
The Web Security Scanner was able to scan all starting URLs and detect the XSS vulnerabilities in Cymbal Bank's application. The ability to automate the detection of these critical vulnerabilities is a major benefit for security-minded organizations like Cymbal Bank. I will now fix the vulnerability in Cymbal Bank's application code and test once again.

### Task 3. Correct the vulnerability and scan again
Now that I have demonstrated Web Security Scanner can detect a XSS vulnerability, I will remediate the vulnerability and run the application scan again.<br>
	1. Return to your SSH window that's connected to your VM instance.<br>
	2. Stop the running application by pressing CTRL + C.<br>
	3. Edit the app.py file using the nano editor by running the following command:<br>
```
nano app.py
```
4. locate the two lines that set the output string:<br>
```
# output_string = "".join([html_escape_table.get(c, c) for c in input_string])
  output_string = input_string
```
5. Remove the ‘#' symbol from the first line and add it to the beginning of the next line (ensure that you indent your code properly!)
final lines must look like the following:<br>
```
@app.route('/output')
def output():
  output_string = "".join([html_escape_table.get(c, c) for c in input_string])
  # output_string = input_string
  return flask.render_template("output.html", output=output_string)
```
6. Now type CTRL+X > Y > Enter to save your changes.<br>
7. Now re-run the application:<br>
```
python3 app.py
```
8. Return to the Google Cloud Console.<br>
9. Click Run at the top of the page.<br>
10. login to the URL http://<EXTERNAL_IP>:8080 using your browser in a separate tab.<br>
11. In the web form enter the same string entered before:<br>
```
<script>alert('This is an XSS Injection')</script>
```
12. Now press the POST button.<br>
13. Verify that this time you see the string displayed in the browser:<br>
14. Return to the Google Cloud Console, where you left off on the Web Security Scanner page.<br>
15. Click Run at the top of the page to re-scan your application.<br>
16. Soon after, you will see that the results yield no more XSS vulnerabilities:
<img src="https://i.imgur.com/QSIjaBg.png?1" height="80%" width="80%" alt="cymbal banking  final scan result1"/><br />
