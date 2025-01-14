# Import the Flask web framework
from flask import Flask

# Create a Flask server and allow interaction using the `app` variable
app = Flask(__name__)

# Define an endpoint which accepts POST requests, reachable from the /record endpoint
@app.route('/record', methods=['POST'])
def record_engine_temperature():
    # Every time the /record endpoint is called, the code in this block is executed
    pass
    # Return a JSON payload and a 200 status code to the client
    return {"success": True}, 200

# Practically identical to the above
@app.route('/collect', methods=['POST'])
def collect_engine_temperature():
    return {"success": True}, 200
