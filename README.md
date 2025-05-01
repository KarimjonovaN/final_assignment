# final_assignment

## (S3, RDS, EC2):
- S3: https://ap-south-1.console.aws.amazon.com/s3/buckets/2t-navruza?region=ap-south-1&bucketType=general&tab=permissions
- EC2: https://ap-south-1.console.aws.amazon.com/ec2/home?region=ap-south-1#InstanceDetails:instanceId=i-036bf066e0a1a25c5
- RDS: https://ap-south-1.console.aws.amazon.com/rds/home?region=ap-south-1#database:id=db-navruza;is-cluster=false

## Script to Import Data:
```
  INSERT INTO tbl_navruza_data (fixed_acidity, residual_sugar, alcohol, density, quality_label) VALUES
    (9.3, 6.4, 13.6, 1.0005, 'high'),
    (11.2, 2.0, 14.0, 0.9912, 'medium'),
    (11.6, 0.9, 8.2, 0.9935, 'low'),
    (12.9, 6.6, 12.7, 1.0002, 'low'),
    (13.9, 13.8, 10.4, 0.9942, 'medium'),
    (12.5, 0.7, 10.5, 0.9933, 'low'),
    (4.3, 9.0, 13.1, 0.9909, 'high'),
    (15.0, 1.7, 12.9, 0.9917, 'high'),
    (12.3, 6.6, 13.2, 0.9936, 'medium');
```


## app.py
```
from flask import Flask, jsonify, request
import psycopg2
from flask_cors import CORS

app = Flask(name)
CORS(app)

conn = psycopg2.connect(
    host='your-rds-endpoint',
    user='admin',
    password='admin1234',
    dbname='winedb'
)

@app.route('/wines', methods=['GET'])
def get_wines():
    with conn.cursor() as cursor:
        cursor.execute("SELECT * FROM tbl_navruza_data ORDER BY id ASC")
        data = cursor.fetchall()
    return jsonify(data)

@app.route('/add', methods=['POST'])
def add_wine():
    data = request.json
    with conn.cursor() as cursor:
        cursor.execute("""
            INSERT INTO tbl_navruza_data (fixed_acidity, residual_sugar, alcohol, density, quality_label)
            VALUES (%s, %s, %s, %s, %s)
        """, (
            data.get('fixed_acidity'),
            data.get('residual_sugar'),
            data.get('alcohol'),
            data.get('density'),
            data.get('quality_label')
        ))
        conn.commit()
    return jsonify({'message': 'Wine entry added successfully'})

@app.route('/delete', methods=['POST'])
def delete_wine():
    wine_id = request.json.get('id')
    with conn.cursor() as cursor:
        cursor.execute("DELETE FROM tbl_navruza_data WHERE id = %s", (wine_id,))
        conn.commit()
    return jsonify({'message': f'Wine entry with id {wine_id} deleted'})

if name == 'main':
    app.run(host='0.0.0.0', port=8000)
```
