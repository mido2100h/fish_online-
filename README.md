# fish_online-
يلا بينا نلعب نصطاد سمك اون لاين
from flask import Flask, jsonify, request

app = Flask(__name__)

# بيانات اللاعب المبدئية
player_data = {
    "username": "Tal",
    "coins": 1000,          # عملات اللعبة العادية
    "gems": 50,             # عملات مشحونة (Premium Currency)
    "level": 1,
    "inventory": {
        "fishing_rods": ["قصبة صيد شاطئ مبتدئ"],
        "boats": [],
        "trucks": [],
        "caught_fish": []
    },
    "restaurant": {
        "owned": False,
        "equipment": []
    }
}

# متجر الشحن والعملات (Monetization Store)
STORE_PACKAGES = [
    {"id": "pack_small", "gems": 100, "price_usd": 0.99, "description": "كيس مجوهرات صغير"},
    {"id": "pack_medium", "gems": 550, "price_usd": 4.99, "description": "صندوق مجوهرات متوسط"},
    {"id": "pack_large", "gems": 1200, "price_usd": 9.99, "description": "خزنة مجوهرات كبيرة"}
]

@app.route('/')
def home():
    return jsonify({"message": "مرحباً بك في خادم لعبة fish_online!"})

# الصفحة الرئيسية لمعلومات اللاعب
@app.route('/api/player', methods=['GET'])
def get_player_profile():
    return jsonify(player_data)

# متجر شراء العملات
@app.route('/api/store', methods=['GET'])
def get_store_packages():
    return jsonify({"store_packages": STORE_PACKAGES})

# عملية شراء عملات (شحن)
@app.route('/api/buy_gems', methods=['POST'])
def buy_gems():
    data = request.get_json()
    package_id = data.get("package_id")
    
    package = next((p for p in STORE_PACKAGES if p["id"] == package_id), None)
    if not package:
        return jsonify({"error": "الحزمة غير موجودة"}), 400
    
    # هنا يتم دمج بوابة الدفع (مثل Stripe / Google Play Billing)
    player_data["gems"] += package["gems"]
    return jsonify({
        "success": True,
        "message": f"تم شحن {package['gems']} مجوهرة بنجاح!",
        "current_gems": player_data["gems"]
    })

# نشاط الصيد (صيد شاطئ أو مركب)
@app.route('/api/fish', methods=['POST'])
def go_fishing():
    data = request.get_json()
    mode = data.get("mode", "shore")  # shore or boat
    
    if mode == "boat" and not player_data["inventory"]["boats"]:
        return jsonify({"error": "يلزم شراء أو تأجير مركب للصيد في الأعماق!"}), 400
    
    # محاكاة صيد سمكة
    fish_type = "سمك بلطي" if mode == "shore" else "سمك وقار فاخر"
    player_data["inventory"]["caught_fish"].append(fish_type)
    
    return jsonify({
        "message": f"تم صيد {fish_type} بنجاح!",
        "caught_fish": player_data["inventory"]["caught_fish"]
    })

# نقل البضائع بالمطاعم وعربات النقل
@app.route('/api/transport', methods=['POST'])
def transport_fish():
    if not player_data["inventory"]["caught_fish"]:
        return jsonify({"error": "لا توجد أسماك لنقلها!"}), 400
    
    num_fish = len(player_data["inventory"]["caught_fish"])
    earned_coins = num_fish * 50
    player_data["coins"] += earned_coins
    player_data["inventory"]["caught_fish"].clear()
    
    return jsonify({
        "message": f"تم نقل الأسماك بعربات النقل إلى المطاعم بنجاح!",
        "earned_coins": earned_coins,
        "total_coins": player_data["coins"]
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
