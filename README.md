from flask import Flask, render_template, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy
from flask_wtf import FlaskForm
from wtforms import StringField, TextAreaField, SubmitField
from wtforms.validators import DataRequired

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your_secret_key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///db.sqlite3'
db = SQLAlchemy(app)

# Database Models
class Resource(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)
    url = db.Column(db.String(200), nullable=True)

class SupportMessage(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    message = db.Column(db.Text, nullable=False)

# Forms
class SupportForm(FlaskForm):
    message = TextAreaField('Your Message', validators=[DataRequired()])
    submit = SubmitField('Send')

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/resources')
def resources():
    resources = Resource.query.all()
    return render_template('resources.html', resources=resources)

@app.route('/mindfulness')
def mindfulness():
    return render_template('mindfulness.html')

@app.route('/support', methods=['GET', 'POST'])
def support():
    form = SupportForm()
    if form.validate_on_submit():
        new_message = SupportMessage(message=form.message.data)
        db.session.add(new_message)
        db.session.commit()
        return redirect(url_for('support'))
    messages = SupportMessage.query.all()
    return render_template('support.html', form=form, messages=messages)

if __name__ == '__main__':
    db.create_all()
    app.run(debug=True)
