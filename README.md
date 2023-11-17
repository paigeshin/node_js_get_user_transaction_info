# node_js_get_user_transaction_info

### Genereate Client Secret 

```swift
const jwt = require("jsonwebtoken");
const fs = require("fs");

function createClientSecret(keyFilePath, teamID, keyID, appID) {
  // Load the private key from a file
  const privateKey = fs.readFileSync(keyFilePath, "utf8");

  // Current time in seconds since epoch
  const currentTime = Math.floor(Date.now() / 1000);

  // JWT header
  const header = {
    alg: "ES256",
    kid: keyID,
  };

  // JWT payload
  const payload = {
    iss: teamID, // Issuer (Team ID)
    iat: currentTime, // Issued At
    exp: currentTime + 15777000, // Expiration Time (6 months)
    aud: "https://appleid.apple.com", // Audience
    sub: appID, // Subject (App ID)
  };

  // Sign the token
  const token = jwt.sign(payload, privateKey, { algorithm: "ES256", header });

  return token;
}

// Usage
const clientSecret = createClientSecret(
  "./data-sharing-key/AuthKey_5PX2J6KFR6.p8", // Path to your private key file
  "64XNY75C9F", // Your 10-character Team ID
  "5PX2J6KFR6", // Your 10-character Key ID
  1619362440 // Your App ID or Services ID
);

console.log("Client Secret: ", clientSecret);

```

### Total Code

```jsx
const jwt = require("jsonwebtoken");
const fs = require("fs");
const axios = require("axios");

const privateKey = fs.readFileSync("./AuthKey_2WC3AN56MF.p8");

// Production URL : https://api.storekit.itunes.apple.com/inApps/v1/transactions/${transactionId}
// Sandbox URL : https://api.storekit-sandbox.itunes.apple.com/inApps/v1/transactions/{transactionId}
async function getTransaction(token, ID) {
  try {
    const response = await axios.get(
      `https://api.storekit.itunes.apple.com/inApps/v1/transactions/${ID}`,
      {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      }
    );
    const signedTransaction = response.data;
    return decode(signedTransaction.signedTransactionInfo);
  } catch (error) {
    console.error(error);
  }
}

function decode(signedTransactionInfo) {
  // Split the JWS transaction into its components
  const [encodedHeader, encodedPayload, signature] =
    signedTransactionInfo.split(".");
  const header = base64UrlDecode(encodedHeader);
  const payload = base64UrlDecode(encodedPayload);
  console.log("Decoded Header: ", header);
  console.log("Decoded Payload: ", payload);
  return { header, payload };
}

// Base64 URL decode the header and payload
function base64UrlDecode(str) {
  const base64 = str.replace(/-/g, "+").replace(/_/g, "/");
  const json = Buffer.from(base64, "base64").toString("utf-8");
  return JSON.parse(json);
}

function generateToken({ privateKey, keyID, issuerID, bundleID }) {
  const header = {
    alg: "ES256",
    kid: keyID, // Key ID
    typ: "JWT",
  };

  const currentTime = Math.floor(Date.now() / 1000); // Current UNIX time in seconds

  const payload = {
    iss: issuerID, // Issuer ID
    iat: currentTime,
    exp: currentTime + 3600, // Expires in 1 hour
    aud: "appstoreconnect-v1",
    bid: bundleID, // Bundle ID
  };

  const token = jwt.sign(payload, privateKey, {
    algorithm: "ES256",
    header: header,
  });
  console.log(token);
  return token;
}

async function main() {
  const token = generateToken({
    privateKey: privateKey,
    keyID: "2WC3AN56MF",
    issuerID: "d429184b-f530-4bd6-b7ad-65ea54289fa4",
    bundleID: "com.munice.MiracleNight",
  });
  console.log(`Token: ${token}`);
  /*
  wvYwxp1OfVaBOtwrPxZMVfEwFq23 => 350001726954946
  kakao:3102784900 => 190001761499075 
  */
  const result = await getTransaction(token, 350001726954946);
  console.log({ result });
}

main();

```
