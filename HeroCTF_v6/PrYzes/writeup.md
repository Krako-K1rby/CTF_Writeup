# PrYzes (web) 458/670 (68.4%)

## Question
A Python enthusiast created a website to distribute prizes, but currently, you are unable to claim them. Discover a method to successfully claim a prize and obtain the flag!

URL: http://web.heroctf.fr:5000/

## Analysis

与えられたURLをクリックすると、次の画面に飛びます。

![PrYzes](PrYzes.png)

まず、この画面のコードがindex.htmlとして与えられているので確認します。

```html
<button id="sendRequestButton" class="my-4 bg-yellow-300 hover:bg-yellow-500 text-zinc text-xl font-bold py-4 px-6 rounded">
    Claim Prizes!
</button>
```

上の部分でボタンのidがsendRequstsButtonであることが分かります。

```html
<script type="text/python">
    from browser import document, ajax, alert
    import hashlib
    import json
    from datetime import datetime

    def on_complete(req):
        json_data = json.loads(req.text)
        if req.status == 200:
            alert(json_data.get("message"))
        else:
            alert(f"Error: {json_data.get('error')}")

    def compute_sha256(data):
        sha256_hash = hashlib.sha256()
        sha256_hash.update(data.encode('utf-8'))
        return sha256_hash.hexdigest()

    def get_current_date():
        current_date = datetime.now().strftime("%d/%m/%Y")
        return current_date

    def send_request(event):
        url = "/api/prizes"
        data = {
            "date": get_current_date()
        }
        json_data = json.dumps(data)
        signature = compute_sha256(json_data)

        req = ajax.ajax()
        req.bind('complete', on_complete)
        req.open('POST', url, True)
        req.set_header('Content-Type', 'application/json')
        req.set_header('X-Signature', signature)
        req.send(json_data)

    document["sendRequestButton"].bind("click", send_request)
</script>
```

そして、ボタンを押したときに呼ばれるsend_request関数では、現在の年月日をjson形式にしたものとそのjsonをSHA256でハッシュ化したものを\
X-Signatureとしてヘッダーに追加し、/api/prizesにPOSTリクエストを送信しています。


また、このwebページの動作は与えられたapp.pyに記述されています。\
注目するべき箇所は/api/prizesにPOSTメソッドでリクエストを送った際の動作です。

#### app.py

```python
@app.route("/api/prizes", methods=["POST"])
def claim_prizes():
    data = request.json
    date_str = data.get("date")
    received_signature = request.headers.get("X-Signature")

    json_data = json.dumps(data)
    expected_signature = compute_sha256(json_data)

    if not received_signature == expected_signature:
        return jsonify({"error": "Invalid signature"}), 400
    
    if not date_str:
        return jsonify({"error": "Date is missing"}), 400

    try:
        date_obj = datetime.strptime(date_str, "%d/%m/%Y")
        if date_obj.year >= 2100:
            return jsonify({"message": FLAG}), 200

        return jsonify({"error": "Please come back later..."}), 400
    except ValueError:
        return jsonify({"error": "Invalid date format"}), 400
```

受け取ったjsonをSHA256でハッシュ化した値とX-Signatureと比較して同じ場合のみにフラグが取得できるようになっています。\
また、受け取った年月日のデータは2100年以降の場合のみにフラグを取得できるようにしています。

## Solution

解析より、送信する年月日の年を2100年以降にして、そのjsonデータをSHA256でハッシュ化した値をX-Signatureにセットした後 \
ボタンを押すことによってフラグを得ることができそうです。\
2100年10月26日のjsonデータのSignatureを求めるsolve.pyを実行し、上記操作を行うことによってフラグを得ました。

### solver.py
```python
import hashlib
import json

def compute_sha256(data):
    sha256_hash = hashlib.sha256()
    sha256_hash.update(data.encode("utf-8"))
    return sha256_hash.hexdigest()

date =  {"date": "26/10/2100"}
date = json.dumps(date)

signature = compute_sha256(date)
print(signature)
```

### Flag
`Hero{PrYzes_4r3_4m4z1ng!!!9371497139}`