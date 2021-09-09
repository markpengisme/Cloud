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
