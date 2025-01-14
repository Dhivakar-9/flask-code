import json
import redis
from flask import Flask, request
from loguru import logger

# Constants
HISTORY_LENGTH = 10
DATA_KEY = "engine_temperature"

# Initialize Flask app
app = Flask(__name__)

# Define the /record endpoint
@app.route('/record', methods=['POST'])
def record_engine_temperature():
    payload = request.get_json(force=True)
    logger.info(f"(*) record request --- {json.dumps(payload)} (*)")

    engine_temperature = float(payload.get("engine_temperature"))
    logger.info(f"engine temperature to record is: {engine_temperature}")

    database = redis.Redis(host="redis", port=6379, db=0, decode_responses=True)
    database.lpush(DATA_KEY, engine_temperature)
    logger.info(f"stashed engine temperature in redis: {engine_temperature}")

    while database.llen(DATA_KEY) > HISTORY_LENGTH:
        database.rpop(DATA_KEY)
    engine_temperature_values = database.lrange(DATA_KEY, 0, -1)
    logger.info(f"engine temperature list now contains these values: {engine_temperature_values}")

    logger.info(f"record request successful")
    return {"success": True}, 200

# Define the /collect endpoint
@app.route('/collect', methods=['POST'])
def collect_engine_temperature():
    database = redis.Redis(host="redis", port=6379, db=0, decode_responses=True)
    engine_temperature_values = database.lrange(DATA_KEY, 0, -1)

    # Convert stored string values to floats
    engine_temperature_values = [float(temp) for temp in engine_temperature_values]

    # Get the most recent value
    current_engine_temperature = engine_temperature_values[0] if engine_temperature_values else None

    # Calculate the average temperature
    average_engine_temperature = (
        sum(engine_temperature_values) / len(engine_temperature_values)
        if engine_temperature_values else None
    )

    logger.info(f"collect request: current={current_engine_temperature}, average={average_engine_temperature}")

    return {
        "current_engine_temperature": current_engine_temperature,
        "average_engine_temperature": average_engine_temperature,
    }, 200
