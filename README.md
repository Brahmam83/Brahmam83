from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://username:password@localhost/database_name'
db = SQLAlchemy(app)

class CodeSubmission(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), nullable=False)
    code_language = db.Column(db.String(50), nullable=False)
    stdin = db.Column(db.String(100))
    source_code = db.Column(db.Text, nullable=False)
    timestamp = db.Column(db.DateTime, default=datetime.utcnow)

@app.route('/submit', methods=['POST'])
def submit_code():
    data = request.json
    new_submission = CodeSubmission(
        username=data['username'],
        code_language=data['code_language'],
        stdin=data['stdin'],
        source_code=data['source_code']
    )
    db.session.add(new_submission)
    db.session.commit()
    return jsonify({'message': 'Code submitted successfully'})

@app.route('/entries', methods=['GET'])
def get_entries():
    entries = CodeSubmission.query.all()
    formatted_entries = [{
        'username': entry.username,
        'code_language': entry.code_language,
        'stdin': entry.stdin,
        'source_code_preview': entry.source_code[:100],
        'timestamp': entry.timestamp.strftime('%Y-%m-%d %H:%M:%S')
    } for entry in entries]
    return jsonify(formatted_entries)

if __name__ == '__main__':
    app.run(debug=True)

    
