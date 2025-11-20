# 🚀 DEPLOYMENT GUIDE
## Zoho Cliqtrix 2025 HR Recruitment Bot

---

## ⚠️ IMPORTANT NOTE

**I cannot directly access your accounts** to deploy the code for you because:
- It requires your personal login credentials
- It involves your private API keys and tokens
- Security best practices prohibit sharing account access

**However**, this guide provides **complete step-by-step instructions** for you to deploy everything yourself.

---

## 📋 PREREQUISITES

Before starting, ensure you have:

✅ **Zoho SalesIQ Account** (with bot builder access)
✅ **Zoho Recruit Account** (with API access enabled)
✅ **Zoho WorkDrive** (enabled on your account)
✅ **Zoho Vault** (for storing credentials securely)
✅ **Twilio Account** (with SMS API enabled)
✅ **Active job postings in Zoho Recruit**

---

## 🎯 DEPLOYMENT STEPS

### PART 1: SET UP ZOHO RECRUIT API

#### Step 1.1: Get OAuth 2.0 Credentials

1. Go to **https://api-console.zoho.com**
2. Click **"Add Client"** → **"Server-based Applications"**
3. Fill in:
   - Client Name: `HR Recruitment Bot`
   - Homepage URL: Your website URL
   - Authorized Redirect URI: `https://www.zoho.com/salesiq`
4. Click **"Create"**
5. **Save these:**
   - Client ID
   - Client Secret

#### Step 1.2: Generate Tokens

1. Generate authorization code:
   ```
   https://accounts.zoho.com/oauth/v2/auth?scope=ZohoRecruit.modules.ALL&client_id=YOUR_CLIENT_ID&response_type=code&access_type=offline&redirect_uri=https://www.zoho.com/salesiq
   ```
2. Visit the URL (replace YOUR_CLIENT_ID)
3. Authorize and copy the **code** from redirect URL
4. Exchange for tokens:
   ```bash
   POST https://accounts.zoho.com/oauth/v2/token
   ```
   Parameters:
   - code: [authorization_code]
   - client_id: [your_client_id]
   - client_secret: [your_client_secret]
   - redirect_uri: https://www.zoho.com/salesiq
   - grant_type: authorization_code

5. **Save the refresh_token** (needed for scripts)

---

### PART 2: SET UP TWILIO

#### Step 2.1: Create Twilio Account

1. Go to **https://www.twilio.com**
2. Sign up and verify your account
3. Get a phone number with SMS capability

#### Step 2.2: Get API Credentials

1. Go to Dashboard
2. Copy:
   - Account SID
   - Auth Token
   - Your Twilio Phone Number

#### Step 2.3: (Optional) Set Up Verify Service

1. Go to **Verify** → **Services**
2. Create new service
3. Copy the **Service SID**

---

### PART 3: CONFIGURE ZOHO VAULT

#### Step 3.1: Store Credentials Securely

1. Open **Zoho Vault**
2. Create a new chamber: **"HR Bot Credentials"**
3. Add secrets:

   **For Zoho Recruit:**
   - Secret Name: `RECRUIT_CLIENT_ID`
   - Secret Name: `RECRUIT_CLIENT_SECRET`
   - Secret Name: `RECRUIT_REFRESH_TOKEN`

   **For Twilio:**
   - Secret Name: `TWILIO_ACCOUNT_SID`
   - Secret Name: `TWILIO_AUTH_TOKEN`
   - Secret Name: `TWILIO_VERIFY_SERVICE_SID` (if using Verify)

---

### PART 4: SET UP ZOHO SALESIQ BOT

#### Step 4.1: Create Bot

1. Login to **Zoho SalesIQ**
2. Go to **Settings** → **Bots**
3. Click **"Create Bot"** → **"Codeless Bot"**
4. Name: `HR Recruitment Assistant`

#### Step 4.2: Create Connections

**Zoho Recruit Connection:**
1. Go to **Settings** → **Developers** → **Connections**
2. Click **"Add Connection"**
3. Select **"OAuth 2.0"**
4. Enter:
   - Connection Name: `zoho_recruit_connection`
   - Authorization URL: `https://accounts.zoho.com/oauth/v2/auth`
   - Token URL: `https://accounts.zoho.com/oauth/v2/token`
   - Scope: `ZohoRecruit.modules.ALL`
   - Client ID: [from Vault]
   - Client Secret: [from Vault]

**Twilio Connection:**
1. Add another connection
2. Connection Name: `twilio_connection`
3. Auth Type: **Basic Auth**
4. Username: Your Account SID
5. Password: Your Auth Token

#### Step 4.3: Import Deluge Scripts

1. In SalesIQ Bot Builder, go to **"Functions"**
2. Create new function: `zoho_recruit_api`
3. Copy the entire code from `scripts/zoho-recruit-api.deluge`
4. Paste and **Save**
5. Create another function: `twilio_otp`
6. Copy code from `scripts/twilio-otp.deluge`
7. Paste and **Save**

---

### PART 5: BUILD BOT FLOWS

#### Flow 1: Welcome & Job Display

1. **Greeting Message:**
   ```
   Welcome to our HR Recruitment Portal! 👋
   I can help you:
   • Browse available jobs
   • Search for specific positions
   • Track your applications
   • Apply for positions
   ```

2. **Call Function:** `fetchActiveJobs()`
3. **Display Carousel:**
   - Loop through jobs
   - Show: Title, Location, Skills, Experience
   - Buttons: [Apply] [Get Details]

#### Flow 2: Job Application Process

**When user clicks "Apply":**

1. **Collect Name:**
   ```
   Great! Let's start your application.
   What's your full name?
   ```

2. **Collect Email:**
   ```
   Please provide your email address:
   ```

3. **Collect Phone:**
   ```
   Please enter your phone number:
   ```
   - Validate using `isValidPhone()`

4. **Send OTP:**
   ```
   Sending verification code to your phone...
   ```
   - Call: `sendOTP(phone_number)`

5. **Verify OTP:**
   ```
   Please enter the 6-digit code:
   ```
   - Call: `verifyOTP(phone_number, entered_code)`
   - If failed: Allow 3 attempts or resend

6. **Resume Upload:**
   ```
   Please upload your resume (PDF/DOC):
   ```
   - Upload to WorkDrive
   - Get file link

7. **Create Candidate:**
   ```deluge
   candidate_data = Map();
   candidate_data.put("First_Name", first_name);
   candidate_data.put("Last_Name", last_name);
   candidate_data.put("Email", email);
   candidate_data.put("Phone", phone);
   candidate_data.put("Resume", resume_link);
   
   candidate_id = createCandidate(candidate_data);
   associateCandidateWithJob(candidate_id, job_id);
   ```

8. **Confirmation:**
   ```
   ✅ Application submitted successfully!
   You'll receive updates via email.
   ```

#### Flow 3: Get Job Details

**When user clicks "Get Details":**

1. Call: `getJobDetails(job_id)`
2. Display:
   - Job Description
   - Responsibilities
   - Requirements
   - Benefits
   - Button: [Apply Now]

#### Flow 4: Smart Job Search

1. **Prompt:**
   ```
   Let's find the perfect job for you!
   ```

2. **Collect Experience:**
   ```
   How many years of experience do you have?
   ```
   - Options: 0-2, 3-5, 6-10, 10+

3. **Collect Skills:**
   ```
   What are your key skills?
   (e.g., Python, Java, React, Marketing)
   ```

4. **Collect Location:**
   ```
   Preferred location?
   ```

5. **Search:**
   ```deluge
   criteria = Map();
   criteria.put("skills", skills);
   criteria.put("location", location);
   matching_jobs = searchJobs(criteria);
   ```

6. Display matching jobs in carousel

#### Flow 5: Application Tracking

1. **Request Email:**
   ```
   Please enter your email to check applications:
   ```

2. **Fetch Data:**
   ```deluge
   candidate = getCandidateByEmail(email);
   applications = getCandidateApplications(candidate_id);
   events = getInterviewEvents(candidate_id);
   ```

3. **Display Applications:**
   - Show list with status
   - If status = "Rejected":
     - Fetch similar jobs
     - Display with [Get Details] [Apply]

4. **Show Upcoming Interviews:**
   ```
   📅 Upcoming Interviews:
   - [Job Title] on [Date] at [Time]
   ```

---

### PART 6: CONFIGURE ZOHO FLOW (for OTP)

#### Step 6.1: Create Flow

1. Go to **Zoho Flow**
2. Create new flow: **"OTP Verification"**

#### Step 6.2: Set Up Webhook

1. **Trigger:** Webhook (from SalesIQ)
2. **Action:** HTTP Request to Twilio
3. **Parse Response:** Return to SalesIQ

---

### PART 7: TESTING

#### Test Checklist:

✅ **Job Display:**
- [ ] Jobs load correctly
- [ ] Carousel navigation works
- [ ] All job details visible

✅ **Application Process:**
- [ ] Form collects all information
- [ ] OTP sends successfully
- [ ] OTP verification works
- [ ] Resume uploads to WorkDrive
- [ ] Candidate created in Recruit
- [ ] Application linked to job

✅ **Job Search:**
- [ ] Search filters work
- [ ] Results match criteria
- [ ] AI suggestions relevant

✅ **Application Tracking:**
- [ ] Email lookup works
- [ ] All applications displayed
- [ ] Status shows correctly
- [ ] Interview events appear
- [ ] Rejected job suggestions shown

---

## 🎯 DEPLOYMENT SUMMARY

**What YOU need to do:**

1. ✅ Get OAuth credentials from Zoho API Console
2. ✅ Get Twilio account and API keys
3. ✅ Store all credentials in Zoho Vault
4. ✅ Create connections in SalesIQ
5. ✅ Copy-paste Deluge scripts into SalesIQ Functions
6. ✅ Build bot flows using Codeless Bot Builder
7. ✅ Test each feature
8. ✅ Deploy to your website

**What I've provided:**

✅ Complete working Deluge scripts (690+ lines)
✅ All API integration code
✅ OAuth 2.0 implementation
✅ OTP verification system
✅ Error handling
✅ This deployment guide

---

## 📞 NEED HELP?

If you encounter issues:

1. **Check Zoho API Console** for quota/limits
2. **Verify OAuth tokens** are not expired
3. **Test Twilio** separately first
4. **Check SalesIQ logs** for errors
5. **Refer to Zoho documentation:**
   - SalesIQ: https://www.zoho.com/salesiq/help/
   - Recruit API: https://www.zoho.com/recruit/developer-guide/

---

## ✅ FINAL CHECK

Before going live:

- [ ] All connections tested
- [ ] OAuth tokens valid
- [ ] OTP working
- [ ] Resume upload functional
- [ ] Job data syncing
- [ ] Error messages user-friendly
- [ ] Bot responses natural
- [ ] Mobile responsive

---

**Your bot is now ready to help candidates find their dream jobs! 🎉**

---

*For Zoho Cliqtrix 2025 Competition*
