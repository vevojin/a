from flask import Flask, request, jsonify
import requests
import re

app = Flask(__name__)

@app.route('/decide', methods=['POST'])
def decide():
    data = request.json

    prompt = f"""
You are a highly skilled volleyball AI.

Your job is to decide whether the player should dive, and if so, which direction, based on:
- Ball velocity
- Ball position relative to player
- Ball movement trend
- Common spiking behavior

Diving is only necessary if the ball is likely to land close and fast. If it's too far or not a threat, return `Wait`.

Choose one of these 9 responses:
FrontDive, BackDive, LeftDive, RightDive, FrontLeftDive, FrontRightDive, BackLeftDive, BackRightDive, Wait

Examples:

Player: 0, 5, -30  
Ball: 10, 8, -42  
Velocity: 3, -6, 5  
Answer: BackRightDive

Player: -5, 5, -12  
Ball: 20, 10, -12  
Velocity: 1, 0, 0  
Answer: Wait

Now answer based on:
Player: {data['player']}  
Ball: {data['ball']}  
Velocity: {data['velocity']}  
If ball is falling: {data['isFalling']}
Ball speed: {data['ballSpeed']}
Distance to player: {data['distance']}
If ball and player are on the same side of the net: {data['sameSideOfNet']}
You may also request for any additional information for more accurate decisions if necessary.
Answer:"""

    response = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": "llama3",
            "prompt": prompt,
            "stream": False
        }
    )

    raw = response.json()['response'].strip()
    print("[RAW]:", raw)

    match = re.search(r"(FrontDive|BackDive|LeftDive|RightDive|FrontLeftDive|FrontRightDive|BackLeftDive|BackRightDive|Wait)", raw)
    decision = match.group(1) if match else "Wait"

    print("[CLEANED]:", decision)
    return jsonify({"decision": decision})

if __name__ == "__main__":
    app.run(port=8000)
