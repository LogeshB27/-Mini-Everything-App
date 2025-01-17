from flask import Flask, request, jsonify
from flask_socketio import SocketIO, join_room, leave_room, send

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret!'
socketio = SocketIO(app, cors_allowed_origins="*")

# In-memory storage for demo purposes
users = {}

@app.route('/register', methods=['POST'])
def register_user():
    data = request.json
    username = data['username']
    if username in users:
        return jsonify({"error": "Username already exists"}), 400
    users[username] = {'messages': []}
    return jsonify({"message": f"User {username} registered successfully."}), 200

@socketio.on('join')
def handle_join(data):
    username = data['username']
    room = data['room']
    join_room(room)
    send(f"{username} has joined the room {room}.", to=room)

@socketio.on('message')
def handle_message(data):
    username = data['username']
    room = data['room']
    message = data['message']
    users[username]['messages'].append(message)
    send(f"{username}: {message}", to=room)

if __name__ == '__main__':
    socketio.run(app, debug=True)

