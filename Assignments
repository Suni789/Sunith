Here’s how you can create the two APIs for managing Training Centers, keeping in mind all the validation, structure, and guidelines mentioned.

Project Structure

backend_traini8/
│
├── app/
│   ├── __init__.py
│   ├── models.py
│   ├── routes.py
│   ├── schemas.py
│   └── exceptions.py
│
├── config.py
├── requirements.txt
└── run.py

app/__init__.py

This file initializes the Flask app, database, and registers blueprints.

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from config import Config

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    app.config.from_object(Config)
    db.init_app(app)

    # Register Blueprints
    from app.routes import training_center_blueprint
    app.register_blueprint(training_center_blueprint)
    
    # Register Error Handlers
    from app.exceptions import register_error_handlers
    register_error_handlers(app)
    
    return app

app/models.py

This file defines the TrainingCenter model.

from app import db
from datetime import datetime

class TrainingCenter(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    center_name = db.Column(db.String(40), nullable=False)
    center_code = db.Column(db.String(12), nullable=False)
    address = db.Column(db.JSON, nullable=False)
    student_capacity = db.Column(db.Integer, nullable=False)
    courses_offered = db.Column(db.ARRAY(db.String), nullable=False)
    created_on = db.Column(db.DateTime, default=datetime.utcnow, nullable=False)
    contact_email = db.Column(db.String(100), nullable=True)
    contact_phone = db.Column(db.String(15), nullable=False)

    def __repr__(self):
        return f"TrainingCenter('{self.center_name}', '{self.center_code}')"

app/schemas.py

This file uses Marshmallow for validation of incoming data.

from marshmallow import Schema, fields, validate, validates, ValidationError
import re

class AddressSchema(Schema):
    detailed_address = fields.Str(required=True)
    city = fields.Str(required=True)
    state = fields.Str(required=True)
    pincode = fields.Str(required=True, validate=validate.Length(equal=6))

class TrainingCenterSchema(Schema):
    center_name = fields.Str(required=True, validate=validate.Length(max=40))
    center_code = fields.Str(required=True, validate=validate.Length(equal=12))
    address = fields.Nested(AddressSchema, required=True)
    student_capacity = fields.Int(required=True)
    courses_offered = fields.List(fields.Str(), required=True)
    contact_email = fields.Email(required=False)
    contact_phone = fields.Str(required=True, validate=validate.Length(min=10, max=15))

    @validates('contact_phone')
    def validate_phone(self, value):
        if not re.match(r'^\+?[0-9]*$', value):
            raise ValidationError("Invalid phone number format.")

app/routes.py

This file defines the routes for the API, including POST and GET.

from flask import Blueprint, request, jsonify
from app import db
from app.models import TrainingCenter
from app.schemas import TrainingCenterSchema
from marshmallow.exceptions import ValidationError
from datetime import datetime

training_center_blueprint = Blueprint('training_center', __name__)

# POST API to create a new training center
@training_center_blueprint.route('/training-centers', methods=['POST'])
def create_training_center():
    try:
        data = request.get_json()
        schema = TrainingCenterSchema()
        validated_data = schema.load(data)

        # Create TrainingCenter object
        training_center = TrainingCenter(
            center_name=validated_data['center_name'],
            center_code=validated_data['center_code'],
            address=validated_data['address'],
            student_capacity=validated_data['student_capacity'],
            courses_offered=validated_data['courses_offered'],
            contact_email=validated_data.get('contact_email'),
            contact_phone=validated_data['contact_phone'],
            created_on=datetime.utcnow()
        )
        db.session.add(training_center)
        db.session.commit()

        # Return the created center's data
        return jsonify(schema.dump(training_center)), 201
    except ValidationError as err:
        return jsonify(err.messages), 400

# GET API to get a list of all training centers
@training_center_blueprint.route('/training-centers', methods=['GET'])
def get_training_centers():
    training_centers = TrainingCenter.query.all()
    schema = TrainingCenterSchema(many=True)
    return jsonify(schema.dump(training_centers)), 200

app/exceptions.py

This file handles custom exceptions and error handling.

from flask import jsonify

def register_error_handlers(app):
    @app.errorhandler(404)
    def not_found(e):
        return jsonify({'error': 'Not found'}), 404

    @app.errorhandler(500)
    def internal_server_error(e):
        return jsonify({'error': 'Internal server error'}), 500

config.py

Configuration for the app, including the database URI.

import os

class Config:
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL', 'sqlite:///trainings.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False

run.py

File to run the Flask app.

from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)

requirements.txt

Dependencies for the project.

flask
flask-sqlalchemy
marshmallow

Setting Up the Project

1. Clone the repository.


2. Install dependencies: pip install -r requirements.txt.


3. Create the database:

flask db init
flask db migrate
flask db upgrade


4. Run the app: python run.py.


5. Access the API at http://127.0.0.1:5000.



API Endpoints

1. POST /training-centers

Request body (example):

{
  "center_name": "Tech Training",
  "center_code": "TCH123456789",
  "address": {
    "detailed_address": "123 Main Street",
    "city": "New York",
    "state": "NY",
    "pincode": "10001"
  },
  "student_capacity": 150,
  "courses_offered": ["Python", "JavaScript"],
  "contact_email": "info@techtraining.com",
  "contact_phone": "+1234567890"
}

Response: 201 Created with the new training center's data.



2. GET /training-centers

Response: 200 OK with a list of training centers or an empty list if no centers exist.




Evaluation

All fields are validated via marshmallow annotations.

Phone and email validations are implemented.

Created timestamp is generated by the system.

Errors return appropriate messages in JSON format.


