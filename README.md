# Miscellaneous utils written in go.

## Have I been pwned?
I have utils based on the data shared by hibp.com.

#### Using `pw/cli/main.go`: 

I stored the dump of hashed pwned passwords in google big query. This cli util takes a password as input and prints the number of times the password has been pwned. This can be run in 2 modes 1) query from big table 2) query from my website. The former requires gcp credentials.

Example usage:

1. Querying big table (needs gcp credentials)

```
$ export GOOGLE_APPLICATION_CREDENTIALS=<credentials_json>
$ export GOOGLE_CLOUD_PROJECT=<project-name>
$ go run pw/cli/main.go -debug=true
Using bigquery as backend
Enter password:
Entered password: wow
Password hash is 8A79FF89C7DBD4655D22C2CE58970514
Password is pwned 6990 times.
Operation took 1.704022925s
```
To not see the entered password, skip the debug flag

2. Making http calls to my website (which queries gcp)

```
$ go run pw/cli/main.go -debug=true -serverAddress=bm.suram.in
Using bm.suram.in for backend.
Enter password:
Entered password: wow
Password hash is 8A79FF89C7DBD4655D22C2CE58970514
====================================
>>>>>Request<<<<
GET /pw/hashed/8A79FF89C7DBD4655D22C2CE58970514 HTTP/1.1
Host: bm.suram.in
<<<<Request>>>>>
>>>>>Response<<<<
HTTP/2.0 200 OK
Content-Security-Policy: default-src * data: 'unsafe-eval' 'unsafe-inline'
Content-Type: application/json
Date: Sun, 03 May 2020 02:42:41 GMT
Referrer-Policy: no-referrer-when-downgrade
Server: nginx
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Vary: Accept-Encoding
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Xss-Protection: 1; mode=block

{"pw_hash":"8A79FF89C7DBD4655D22C2CE58970514","pwn_count":6990,"meta":{"op_time":"619.592825ms"}}
<<<<Response>>>>>
====================================
Password is pwned 6990 times.
Operation took 1.296939138s
```
In debug mode, I pring the http request and response made to the server (my server). To skip this dont specify set debug to false.

#### `server.go`
This is the webserver that runs at [bm.suram.in/pw](http://bm.suram.in/pw). This needs gcp credentials.

## TOTP 
Implementation of TOTP (used by Authenticator apps;  RFC 6238).

The totp.go file has 3 tasks

1. `test`: Tests the implementation. This does the following 1) creates a TOTP secret 2) Serializes it in QR code in /tmp 3) Loads the QR code as another TOTP object 4) Compares the otps from the original TOTP and the one loaded from QR code.

It compares the OTP generated by this code with the one generated by github.com/gokyle/hotp
```
$ go run totp/totp.go
Will save the qr to /tmp/totp_DALOV54DZFI6DRMT.png
totp1: main.TOTP{issuer:"test_totp.com", digits:6, secret:"KFZPBW6UQDU2NAQ7UKZHFCJTUFPKBNIK", note:"user_DALOV54DZFI6DRMT"}
totp2: main.TOTP{issuer:"test_totp.com", digits:6, secret:"KFZPBW6UQDU2NAQ7UKZHFCJTUFPKBNIK", note:"user_DALOV54DZFI6DRMT"}
OTP       : 898992 898992
GokyleOTP : 898992 898992
```

2. `create`: Creates a TOTP secret, stores it as a qr image in the given path.
```
$ go run totp/totp.go --task=create -create_qr_path=/tmp/flowerpot.png -create_user=flower -create_digits=7 -create_issuer=dotslash
Stored totp qr at /tmp/flowerpot.png
Current totp: 7777079
```

3. `otp`: Gets the otp from a TOTP url/qr image path.
```
$ go run totp/totp.go -debug=true --task=otp -otp_qr_path /tmp/flowerpot.png
URL: otpauth://totp/flower?issuer=dotslash&digits=7&secret=NKEDE5UCHJ3FZWWA42FZ2CTLMEQUW4RY
time Seq: 52949166
OTP took 19.622µs
$ go run totp/totp.go -debug=true --task=otp -otp_url 'otpauth://totp/flower?issuer=dotslash&digits=7&secret=NKEDE5UCHJ3FZWWA42FZ2CTLMEQUW4RY'
URL: otpauth://totp/flower?issuer=dotslash&digits=7&secret=NKEDE5UCHJ3FZWWA42FZ2CTLMEQUW4RY
time Seq: 52949170
OTP took 13.784µs
3637532
```
