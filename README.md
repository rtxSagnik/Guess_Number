
from flask import Flask, render_template, request, session, redirect, url_for
import random

app = Flask(__name__)
app.secret_key = "your_secret_key"

MAX_TRIES = 8

@app.route('/', methods=['GET', 'POST'])
def index():
    if 'number' not in session:
        return redirect(url_for('choose_difficulty'))

    message = ""
    max_num = session.get('max_num', 100)
    tries = session.get('tries', 0)

    if request.method == 'POST':
        guess = int(request.form['guess'])

        if guess < 1 or guess > max_num:
            message = f"Please enter a number between 1 and {max_num}."
        else:
            session['tries'] += 1
            tries = session['tries']

            if guess < session['number']:
                message = "Too low! Try a Higher Number"
            elif guess > session['number']:
                message = "Too high! Try a Lower Number"
            else:
                message = "🎉 Correct! You guessed the number!"
                return render_template('index.html', message=message, max_num=max_num)

            if tries >= MAX_TRIES:
                message = f"❌ Game over! The number was {session['number']}."
                return render_template('index.html', message=message, max_num=max_num)

    return render_template('index.html', message=message, max_num=max_num)


@app.route('/choose', methods=['GET', 'POST'])
def choose_difficulty():
    if request.method == 'POST':
        difficulty = request.form['difficulty']
        session['difficulty'] = difficulty

        if difficulty == 'easy':
            session['max_num'] = 10
        elif difficulty == 'medium':
            session['max_num'] = 50
        else:
            session['max_num'] = 100

        session['number'] = random.randint(1, session['max_num'])
        session['tries'] = 0

        return redirect(url_for('index'))

    return render_template('difficulty.html')


@app.route('/exit')
def exit_game():
    session.clear()
    return render_template('exit.html')


if __name__ == "__main__":
    app.run(debug=True)
