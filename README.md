# Ravshanbek's Web Application Project

## Overview

This project is a simple full-stack web application built using HTML, JavaScript, Flask (Python), and PostgreSQL. It allows users to add and delete text entries from a PostgreSQL database via a user-friendly web interface.

* **Frontend**: Hosted on S3 using `index_ravshanbek.html`
* **Backend**: A Flask app running on an EC2 instance
* **Database**: PostgreSQL hosted on AWS RDS

## Project Structure

```
project/
├── index_ravshanbek.html       # Frontend HTML file (hosted on S3)
├── app.py                      # Flask backend (deployed on EC2)
├── venv/                       # Python virtual environment
└── README.md                   # This file
```

## How to Run the Application

### 1. RDS (PostgreSQL) Setup

* RDS endpoint: `ravshanbek-2t-s3web.ct6ei6agkus4.ap-south-1.rds.amazonaws.com`
* Database name: `ravshanbek_2t_s3web`
* Username: `postgres`
* Password: `Ravbex_1825`

#### Connect to the Database:

```bash
psql -h ravshanbek-2t-s3web.ct6ei6agkus4.ap-south-1.rds.amazonaws.com -U postgres -d ravshanbek_2t_s3web
```

#### View Table Data:

```sql
SELECT * FROM test_table;
```
#### Delete column
```sql
ALTER TABLE test_table DROP COLUMN text;
```
#### Add column
```sql
ALTER TABLE test_table ADD COLUMN text TEXT;
```
#### Drop a table
```sql
DROP TABLE test_table;
```
#### Rename table name
```sql
ALTER TABLE old_table_name RENAME TO new_table_name;
```
#### Change password table
```sql
ALTER USER postgres WITH PASSWORD 'NewSecurePassword';
```

### 2. Flask Backend on EC2

#### Step-by-step:

```bash
# SSH into your EC2 instance
ssh -i your-key.pem ubuntu@<EC2-Public-IP>

# Navigate to your project folder
cd ~/webapp_ravshanbek

# Activate your virtual environment
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run Flask app
python3 app.py
```

Ensure Flask runs on `0.0.0.0`:

```python
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### 3. Static Hosting on S3

* Upload `index_ravshanbek.html` to your S3 bucket
* Enable static website hosting in the S3 bucket properties
* Set permissions to make the file public (or configure CloudFront)

#### HTML JavaScript Points to EC2

```javascript
const apiBaseUrl = 'http://<EC2-Public-IP>:5000';
```

## Flask App Code (app.py)

```python
from flask import Flask, request, jsonify
from flask_cors import CORS
import psycopg2

app = Flask(__name__)
CORS(app)

# PostgreSQL RDS connection details
conn = psycopg2.connect(
    host="ravshanbek-2t-s3web.ct6ei6agkus4.ap-south-1.rds.amazonaws.com",
    database="ravshanbek_2t_s3web",
    user="postgres",
    password="Ravbex_1825"
)

@app.route('/')
def home():
    return "Flask app is running!"

@app.route('/add', methods=['POST'])
def add_data():
    data = request.json.get('text')
    cur = conn.cursor()
    try:
        cur.execute("INSERT INTO test_table (text) VALUES (%s);", (data,))
        conn.commit()
        return jsonify({"message": "Data added"}), 200
    except Exception as e:
        conn.rollback()
        return jsonify({"error": str(e)}), 500
    finally:
        cur.close()

@app.route('/delete', methods=['POST'])
def delete_data():
    data = request.json.get('text')
    cur = conn.cursor()
    try:
        cur.execute("DELETE FROM test_table WHERE text = %s;", (data,))
        conn.commit()
        return jsonify({"message": "Data deleted"}), 200
    except Exception as e:
        conn.rollback()
        return jsonify({"error": str(e)}), 500
    finally:
        cur.close()

@app.route('/list', methods=['GET'])
def list_data():
    cur = conn.cursor()
    try:
        cur.execute("SELECT text FROM test_table ORDER BY id DESC;")
        rows = cur.fetchall()
        return jsonify([row[0] for row in rows])
    except Exception as e:
        return jsonify({"error": str(e)}), 500
    finally:
        cur.close()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

## HTML Code (index\_ravshanbek.html)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Ravshanbek's Web App</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
    <h1>Database Controller</h1>

    <input type="text" id="textInput" placeholder="Enter some text">
    <br><br>

    <button onclick="addData()">Add</button>
    <button onclick="deleteData()">Delete</button>

    <p id="response"></p>

    <h2>Data in Table</h2>
    <ul id="dataList"></ul>

    <script>
        const apiBaseUrl = 'http://13.235.87.170:5000'; // Replace with your EC2 IP

        function addData() {
            const text = document.getElementById('textInput').value;

            fetch(`${apiBaseUrl}/add`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ text: text })
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('response').innerText = data.message || data.error;
                fetchData(); // Refresh data
            })
            .catch(error => console.error('Error:', error));
        }

        function deleteData() {
            const text = document.getElementById('textInput').value;

            fetch(`${apiBaseUrl}/delete`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ text: text })
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('response').innerText = data.message || data.error;
                fetchData(); // Refresh data
            })
            .catch(error => console.error('Error:', error));
        }

        function fetchData() {
            fetch(`${apiBaseUrl}/list`)
                .then(response => response.json())
                .then(data => {
                    const dataList = document.getElementById('dataList');
                    dataList.innerHTML = ''; // Clear old list
                    data.forEach(item => {
                        const li = document.createElement('li');
                        li.textContent = item;
                        dataList.appendChild(li);
                    });
                })
                .catch(error => {
                    document.getElementById('dataList').innerHTML = '<li>Error loading data</li>';
                    console.error('Error fetching data:', error);
                });
        }

        // Load data on page load
        window.onload = fetchData;
    </script>
</body>
</html>
```

## Links to Deployed Resources

* **S3 Static Site**: `http://ravshanbek-2t-final.s3-website.ap-south-1.amazonaws.com`
* **EC2 Flask App**: `http://13.235.87.170:5000`
* **RDS Endpoint**: `ravshanbek-2t-s3web.ct6ei6agkus4.ap-south-1.rds.amazonaws.com`

---

Change EC2 security group which allows inbound traffic on port 5000, and your RDS instance allows access from the EC2's security group or public IP.

## License
Ravshanbek Zokirjonov

05.05.2025
