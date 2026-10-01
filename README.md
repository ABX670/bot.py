from flask import Flask, request, jsonify
from config import OWNER_NUMBER
import database
from commands.admin import handle_admin, is_admin
from commands.games import handle_games
from commands.fun import handle_fun
from commands.general import handle_general

app=Flask(__name__)
db=database.load()

@app.route("/webhook", methods=["POST"])
def webhook():
    data=request.json
    sender=data.get("sender"); text=data.get("text",""); mentions=data.get("mentions",[])
    group_members=data.get("group_members",[sender])
    if not db["settings"]["bot_enabled"] and text.strip()!="تشغيل":
        return jsonify({"reply":None})
    reply=None
    if is_admin(db,sender):
        reply=handle_admin(db,sender,text,mentions)
    if not reply:
        reply=handle_games(db,sender,text)
    if not reply and text.strip() in ["تزويج","خطف"] and is_admin(db,sender):
        reply=handle_fun(db,sender,text,group_members)
    if not reply:
        reply=handle_general(db,sender,text,mentions)
    database.save(db)
    return jsonify({"reply":reply})

@app.route("/", methods=["GET"])
def home(): return "WhatsApp Group Bot is running!"

if __name__=="__main__":
    app.run(port=5000)
