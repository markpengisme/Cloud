# Build Apps & Websites with Firebase

## Importing Data to a Firestore Database

- In this lab you will upload the existing data to a Cloud Firestore database.

- ![Architecture](https://cdn.qwiklabs.com/Yfo0T7MHSB8V2VwDmVYNMJQo5bly1%2BtEbv%2FBrpUNbZ8%3D)

- Set up Firestore in Google Cloud

  - **Navigation menu** > **Firestore** > **Select Native Mode**  > **Select a location** > **Create Database**

- Write database import code

  ```sh
  git clone https://github.com/rosera/pet-theory
  cd pet-theory/lab01
  npm install @google-cloud/firestore
  npm install @google-cloud/logging
  npm install faker
  ```

  - importTestData.js(updated)

    ```js
    const {promisify} = require('util');
    const parse       = promisify(require('csv-parse'));
    const {readFile}  = require('fs').promises;
    const {Firestore} = require('@google-cloud/firestore');
    const {Logging} = require('@google-cloud/logging');
    const logName = 'pet-theory-logs-importTestData';
    
    // Creates a Logging client
    const logging = new Logging();
    const log = logging.log(logName);
    
    const resource = {
      type: 'global',
    };
    
    if (process.argv.length < 3) {
      console.error('Please include a path to a csv file');
      process.exit(1);
    }
    
    const db = new Firestore();
    
    function writeToFirestore(records) {
      const batchCommits = [];
      let batch = db.batch();
      records.forEach((record, i) => {
        var docRef = db.collection('customers').doc(record.email);
        batch.set(docRef, record);
        if ((i + 1) % 500 === 0) {
          console.log(`Writing record ${i + 1}`);
          batchCommits.push(batch.commit());
          batch = db.batch();
        }
      });
      batchCommits.push(batch.commit());
      return Promise.all(batchCommits);
    }
    
    function writeToDatabase(records) {
      records.forEach((record, i) => {
        console.log(`ID: ${record.id} Email: ${record.email} Name: ${record.name} Phone: ${record.phone}`);
      });
      return ;
    }
    
    async function importCsv(csvFileName) {
      const fileContents = await readFile(csvFileName, 'utf8');
      const records = await parse(fileContents, { columns: true });
      try {
        await writeToFirestore(records);
      }
      catch (e) {
        console.error(e);
        process.exit(1);
      }
      console.log(`Wrote ${records.length} records`);
      
      // A text log entry
      success_message = `Success: importTestData - Wrote ${records.length} records`
      const entry = log.entry({resource: resource}, {message: `${success_message}`});
      log.write([entry]);
    }
    
    importCsv(process.argv[2]).catch(e => console.error(e));
    
    ```

  - createTestData.js(updated)

    ```js
    const fs = require('fs');
    const faker = require('faker');
    const {Logging} = require('@google-cloud/logging');        //add this
    const logName = 'pet-theory-logs-createTestData';
    
    // Creates a Logging client
    const logging = new Logging();
    const log = logging.log(logName);
    
    const resource = {
      // This example targets the "global" resource for simplicity
      type: 'global',
    };
    
    function getRandomCustomerEmail(firstName, lastName) {
      const provider = faker.internet.domainName();
      const email = faker.internet.email(firstName, lastName, provider);
      return email.toLowerCase();
    }
    
    async function createTestData(recordCount) {
      const fileName = `customers_${recordCount}.csv`;
      var f = fs.createWriteStream(fileName);
      f.write('id,name,email,phone\n')
      for (let i=0; i<recordCount; i++) {
        const id = faker.random.number();
        const firstName = faker.name.firstName();
        const lastName = faker.name.lastName();
        const name = `${firstName} ${lastName}`;
        const email = getRandomCustomerEmail(firstName, lastName);
        const phone = faker.phone.phoneNumber();
        f.write(`${id},${name},${email},${phone}\n`);
      }
      console.log(`Created file ${fileName} containing ${recordCount} records.`);
      
      // A text log entry
      const success_message = `Success: createTestData - Created file ${fileName} containing ${recordCount} records.`
      const entry = log.entry({resource: resource}, {name: `${fileName}`, recordCount: `${recordCount}`, message: `${success_message}`});
      log.write([entry]);
    }
    
    recordCount = parseInt(process.argv[2]);
    if (process.argv.length != 3 || recordCount < 1 || isNaN(recordCount)) {
      console.error('Include the number of test data records to create. Example:');
      console.error('    node createTestData.js 100');
      process.exit(1);
    }
    
    createTestData(recordCount);
    ```

  - Create test data

    ```sh
    PROJECT_ID=$(gcloud config get-value project)
    gcloud config set project $PROJECT_ID
    node createTestData 1000
    cat customers_1000.csv
    ```

- Import the test customer data

  ```sh
  node importTestData customers_1000.csv
  ```

- Inspect the data in Firestore

  - **Navigation menu** > **Firestore** > **Data**

- Edit and delete data from Firestore

  - Edit: Hover over the field to update and delete
  - Delete: Click the three vertically placed dots next to **customers** >  **Delete collection** > **Delete**

- Add a developer to the project without giving them Firestore access

  - [understanding roles](https://cloud.google.com/iam/docs/understanding-roles#predefined_roles) 

    - search for "view logs" > `roles/logging.viewer`
    - search for "repository" > `roles/source.writer`

    ```sh
    ## 2nd user as the developer, so replace [EMAIL] to Username 2
    gcloud projects add-iam-policy-binding $PROJECT_ID --member=user: --role=roles/logging.viewer
    gcloud projects add-iam-policy-binding $PROJECT_ID --member=user: --role=roles/source.writer
    ```

- Login username2

  - Loggin: OK
  - Source Repositories: OK
  - Filestore: No permission

## Build a Serverless Web App with Firebase

- In the previous lab, Migrating Data to a Firestore Database, you learned how to leverage [Firestore](https://firebase.google.com/docs/firestore) to host customer data.

- In this lab you will build a fully fledged Firebase web app that allows users to log information and schedule appointments in real time.

- The primary benefit of Firebase hosting is that it is serverless, so there is no infrastructure to manage.

- ![arch](https://cdn.qwiklabs.com/6pi71cGa4Nz1yhgU%2Biwp5cDD7Mv2JLuaIIHmW448Xiw%3D)

- Enable the Google Cloud Firestore API

  - **Navigation menu** > **APIs & Services** > **Library** > `Google Cloud Firestore API` > **Enable**
  - **Navigation menu** > **Firestore** > **Select Native Mode** > Select **nam5 (United States)**  >  **Create Database** >

- Create a Firebase project

  - Open [Firebase console](https://console.firebase.google.com/) > **Create project** > Select Qwiklabs Project ID > **Accept** & **Continue** > Confirm Pay as you go plan >
  - **Continue** >
  - Disable **Google Analytics for your Firebase Project** >  **Add Firebase** > **Continue** 

- Register your app

  - Select the **web icon** > `Pet Theory` > Check  **Set up Firebase hosting** > **Register app** 
  - **Next** > **Next** > **Continue to console**

- Install the Firebase CLI and deploy to Firebase Hosting

  ```sh
  git clone https://github.com/rosera/pet-theory.git
  cd pet-theory/lab02
  npm init --yes
  npm install -g firebase-tools
  firebase login --no-localhost
  Y
  # Click the URL and Get the access code
  [access code]
  
  gcloud config set compute/region us-central1
  firebase init
  ```

  ```text
  === Project Setup
  
  First, let's associate this project directory with a Firebase project.
  You can create multiple project aliases by running firebase use --add,
  but for now we'll just set up a default project.
  
  i  Using project qwiklabs-gcp-03-xxxxxxxxxxxx (qwiklabs-gcp-03-xxxxxxxxxxxx)
  
  === Firestore Setup
  
  Firestore Security Rules allow you to define how and when to allow
  requests. You can keep these rules in your project directory
  and publish them with firebase deploy.
  
  ? What file should be used for Firestore Rules? firestore.rules
  ? File firestore.rules already exists. Do you want to overwrite it with the Firestore Rules from the Firebase Console? No
  
  Firestore indexes allow you to perform complex queries while
  maintaining performance that scales with the size of the result
  set. You can keep index definitions in your project directory
  and publish them with firebase deploy.
  
  ? What file should be used for Firestore indexes? firestore.indexes.json
  ? File firestore.indexes.json already exists. Do you want to overwrite it with the Firestore Indexes from the Firebase Console? Yes
  
  === Hosting Setup
  
  Your public directory is the folder (relative to your project directory) that
  will contain Hosting assets to be uploaded with firebase deploy. If you
  have a build process for your assets, use your build's output directory.
  
  ? What do you want to use as your public directory? public
  ? Configure as a single-page app (rewrite all urls to /index.html)? No
  ? Set up automatic builds and deploys with GitHub? No
  ? File public/404.html already exists. Overwrite? No
  i  Skipping write of public/404.html
  ? File public/index.html already exists. Overwrite? No
  i  Skipping write of public/index.html
  
  i  Writing configuration info to firebase.json...
  i  Writing project information to .firebaserc...
  i  Writing gitignore file to .gitignore...
  ```

- Set up authentication and a database

  - **Firebase Console** > **Project Overview** > **Authentication** >  **Get Started**
  - **Sing-in method** tab > enable **Google** > In the **Project support email** Select Qwiklabs Google account > **Save**

- Configure Firestore authentication and add sign-in to your web app

  - **Open editor **>  `pet-theory/lab02/firestore.rules`

    This will configure the Firestore database so that each user can only access their own data.

    ```sh
    service cloud.firestore {	match /databases/{database}/documents {        match /customers/{email} {            allow read, write: if request.auth.token.email == email;        }        match /customers/{email}/{document=**} {            allow read, write: if request.auth.token.email == email;        }	}}
    ```

- Deploy your application

  ```sh
  cd ~/pet-theory/lab02/firebase deploy# Click the Hosting URL and Sign in with Google
  ```

  - You have now deployed code to let users use Google authentication to access the appointments app.

    > **Note:** Managing passwords is a difficult task and could expose your company to additional risk. Also, users don't want to create yet another user id and password. A small company like Pet Theory doesn't have the resources or requisite skill set to do this. In this instance it is much better to let the application users log in with their existing Google account (or any other identity providers)!

- Add a customer page to your web app

  - `public/customer.js`

    ```js
    let user;firebase.auth().onAuthStateChanged(function(newUser) {  user = newUser;  if (user) {    const db = firebase.firestore();    db.collection("customers").doc(user.email).onSnapshot(function(doc) {      const cust = doc.data();      if (cust) {        document.getElementById('customerName').setAttribute('value', cust.name);        document.getElementById('customerPhone').setAttribute('value', cust.phone);      }      document.getElementById('customerEmail').innerText = user.email;    });  }});document.getElementById('saveProfile').addEventListener('click', function(ev) {  const db = firebase.firestore();  var docRef = db.collection('customers').doc(user.email);  docRef.set({    name: document.getElementById('customerName').value,    email: user.email,    phone: document.getElementById('customerPhone').value,  })})
    ```

  - `public/style.css`

    ```css
    body { background: #ECEFF1; color: rgba(0,0,0,0.87); font-family: Roboto, Helvetica, Arial, sans-serif; margin: 0; padding: 0; }#message { background: white; max-width: 360px; margin: 100px auto 16px; padding: 32px 24px 16px; border-radius: 3px; }#message h3 { color: #888; font-weight: normal; font-size: 16px; margin: 16px 0 12px; }#message h2 { color: #ffa100; font-weight: bold; font-size: 16px; margin: 0 0 8px; }#message h1 { font-size: 22px; font-weight: 300; color: rgba(0,0,0,0.6); margin: 0 0 16px;}#message p { line-height: 140%; margin: 16px 0 24px; font-size: 14px; }#message a { display: block; text-align: center; background: #039be5; text-transform: uppercase; text-decoration: none; color: white; padding: 16px; border-radius: 4px; }#message, #message a { box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24); }#load { color: rgba(0,0,0,0.4); text-align: center; font-size: 13px; }@media (max-width: 600px) {  body, #message { margin-top: 0; background: white; box-shadow: none; }  body { border-top: 16px solid #ffa100; }}
    ```

  - Cloud shell `firebase deploy`

  - Hard Refresh(**CMND+SHIFT+R (Mac) or CTRL+SHIFT+R (Windows)**) the web app page

    - Name: `student`
    - Password: `Pass.123`
    - **Save profile**

  - **Firebase Console** > **Firestore** > Check the profile saved

  - Return to the web app page and click on the **Appointments** link. 

- Let users schedule appointments

  - `public/appointments.html`

    ```html
    <!DOCTYPE html><html>  <head>    <meta charset="utf-8">    <meta name="viewport" content="width=device-width, initial-scale=1">    <title>Pet Theory appointments</title>    <script src="/__/firebase/6.4.2/firebase-app.js"></script>    <script src="/__/firebase/6.4.2/firebase-auth.js"></script>    <script src="/__/firebase/6.4.2/firebase-firestore.js"></script>    <script src="/__/firebase/init.js"></script>    <link type="text/css" rel="stylesheet" href="styles.css" />  </head>  <body>    <div id="message">      <h2>Scheduled appointments</h2>      <div id="appointments"></div>      <hr/>      <h2>Schedule a new appointment</h2>      <select id="timeslots">        <option value="0">Choose time</option>      </select>      <br><br>      <button id="makeAppointment">Schedule</button>    </div>    <script src="appointments.js"></script>  </body></html>
    ```

  - `public/appointments.js`

    ```js
    let user;firebase.auth().onAuthStateChanged(function(newUser) {  user = newUser;  if (user) {    const db = firebase.firestore();    const appColl = db.collection('customers').doc(user.email).collection('appointments');    appColl.orderBy('time').onSnapshot(function(snapshot) {      const div = document.getElementById('appointments');      div.innerHTML = '';      snapshot.docs.forEach(appointment => {        div.innerHTML += formatDate(appointment.data().time) + '<br/>';      })      if (div.innerHTML == '') {        div.innerHTML = 'No appointments scheduled';      }    });  }});const timeslots = document.getElementById('timeslots');getOpenTimes().forEach(time => {  timeslots.add(new Option(formatDate(time), time));});document.getElementById('makeAppointment').addEventListener('click', function(ev) {  const millis = parseInt(timeslots.selectedOptions[0].value);  if (millis > 0) {    const db = firebase.firestore();    db.collection('customers').doc(user.email).collection('appointments').add({      time: millis    })    timeslots.remove(timeslots.selectedIndex);  }})function getOpenTimes() {  const retVal = [];  let startDate = new Date();  startDate.setMinutes(0);  startDate.setSeconds(0);  startDate.setMilliseconds(0);  let millis = startDate.getTime();  while (retVal.length < 5) {    const hours = Math.floor(Math.random() * 5) + 1;    millis += hours * 3600 * 1000;    if (isDuringOfficeHours(millis)) {      retVal.push(millis);    }  }  return retVal;}function isDuringOfficeHours(millis) {  const aDate = new Date(millis);  return aDate.getDay() != 0 && aDate.getDay() != 6 &&         aDate.getHours() >= 9 && aDate.getHours() <= 17;}function formatDate(millis) {  const aDate = new Date(millis);  const days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];  const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',                  'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];  return days[aDate.getDay()] + ' ' + aDate.getDate() + ' ' +         months[aDate.getMonth()] + ', ' + aDate.getHours() + ':' +         (aDate.getMinutes() < 10? '0'+aDate.getMinutes(): aDate.getMinutes());}
    ```

  - Cloud shell `firebase deploy`

  - Refresh the web app page.

  - Schedule a couple of appointments

  - **Firebase Console** > **Firestore** > Click **appointments** > Check the appointments time  saved

- See Firestore Real-Time updates

  - **Firebase Console** > **Hosting** 
  - Click **xxxx-web.app**, then Sign in with Google
  - Click **xxxx-firebaseapp.com**, then Sign in with Google
  - Arrange the two browser tabs side-by-side
  - Schedule a new appointment in any tabs.
  - Appointment  automatically added without having to refresh
  - Try edit and delete data in the **Firebase Console** > **Firestore** > **appointments**
