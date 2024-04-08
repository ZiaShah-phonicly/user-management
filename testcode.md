from flask import Flask, request, jsonify
from pymongo import MongoClient

app = Flask(__name__)
client = MongoClient('mongodb://localhost:27017/')
db = client['user_management']
users_collection = db['users']

# User Registration
@app.route('/register', methods=['POST'])
def register_user():
    user_data = request.json
    if 'username' not in user_data or 'email' not in user_data or 'password' not in user_data:
        return jsonify({'error': 'Incomplete user data'}), 400
    
    if users_collection.find_one({'email': user_data['email']}):
        return jsonify({'error': 'Email already exists'}), 400
    
    users_collection.insert_one(user_data)
    return jsonify({'message': 'User registered successfully'}), 201

# User Login
@app.route('/login', methods=['POST'])
def login_user():
    login_data = request.json
    if 'email' not in login_data or 'password' not in login_data:
        return jsonify({'error': 'Incomplete login data'}), 400
    
    user = users_collection.find_one({'email': login_data['email'], 'password': login_data['password']})
    if not user:
        return jsonify({'error': 'Invalid email or password'}), 401
    
    return jsonify({'message': 'Login successful'}), 200

# Get User Profile
@app.route('/profile/<username>', methods=['GET'])
def get_user_profile(username):
    user = users_collection.find_one({'username': username})
    if not user:
        return jsonify({'error': 'User not found'}), 404
    
    return jsonify(user), 200

# Update User Profile
@app.route('/profile/<username>', methods=['PUT'])
def update_user_profile(username):
    updated_data = request.json
    if not updated_data:
        return jsonify({'error': 'No data provided'}), 400
    
    users_collection.update_one({'username': username}, {'$set': updated_data})
    return jsonify({'message': 'User profile updated successfully'}), 200

if __name__ == '__main__':
    app.run(debug=True)
