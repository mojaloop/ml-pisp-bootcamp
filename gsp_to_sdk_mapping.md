# GSP API to Thirdparty API mapping

PISP issues quotation request.
```
POST /v3/getTransferFundsQuotationRequest

{
  "requestHeader": {
    "protocolVersion": {
      "major": 3
    },
    "requestId": "bWVyY2hhbnQgdHJhbnN",
    "requestTimestamp": {
      "epochMillis": "1502220196077"
    },
    "paymentIntegratorAccountId": "Mojaloop"
  },
  // This is not listed in the draft document, but for the purpose of the demo
  // we need the payer's thirdparty link identifier. This token seemed to be the most
  // suitable object in the GSP Api, so we are adding this here to the quotation request.
  "googlePaymentToken": {
    "issuerId": {
      "value": "Mojaloop"
    },
    "token": "payer@testbank"
  },
  "associationId": "ZXhhbXBsZSB1bmlxdWUgcGF5bWVudC",
  "transactionDescription": "Payment",
  "amount": {
    "amountMicros": "5000000",
    "currencyCode": "SGD"
  },
  "payeeProxy": {
    "payeeProxyKey": {
      "networkId": "paynow",
      "phoneNumber": "+6577778888"
    },
    "payeeProxyLookupRequestId": "bWVyY2hhbnQgdHJhbnN"
  }
}
```

Core connector takes POST /v3/getTransferFundsQuotationRequest and performs thirdparty party lookup. It then uses both the POST /v3/getTransferFundsQuotationRequest and party lookup response to initiate thirdparty transaction.

```
POST /thirdpartyTransaction/partyLookup
{
  transactionRequestId: convert_to_uuid(getTransferFundsQuotationRequest.associationId) || uuidv5(getTransferFundsQuotationRequest.associationId, config.namespace);,
  payee: {
    partyIdType: "MSISDN",
    partyIdentifier: "+6577778888"
  }
}

Response

{
  body: {
    "party": {
      "partyIdInfo": {
        "partyIdType": "MSISDN
        "partyIdentifier": "+6577778888"
        "fspId": "receiverFsp"
      },
      "personalInfo": {
        "complexName": {
          "firstName": "Bob"
          "middleName": "O"
          "lastName": "Babirusa"
        },
        "dateOfBirth": "1970-01-01"
      },
      "name": "Bob Babirusa"
    },
    "currentState": "partyLookupSuccess"
  }
}
```

```
POST /thirdpartyTransactions/{transactionRequestId}/initiate
{
  amount: {
    amount: "convert_to_fspiop_currency(getTransferFundsQuotationRequest.amount.amountMicros)",
    currency: "getTransferFundsQuotationRequest.currencyCode"
  },
  "amountType": "SEND",
  "payee": {
    ...partyLookupResponse.body.party
  },
  "payer": {
    "partyIdType": "THIRD_PARTY_LINK",
    "partyIdentifier": getTransferFundsQuotationRequest.googlePaymentToken.token,
  },
  "transactionType": {
    "scenario": "TRANSFER",
    "initiator": 'PAYER",
    "initiatorType": "CONSUMER"
  },
  expiration: new Date(now() + config.expirationTimeout)
}

response: {
  body: {
    "authorization": {
      "transactionRequestId": "c2470148-1be2-4c0b-aece-aa8dcb92a6cc",
      "transferAmount": {
        "amount": "5.05",
        "currency": "SGD"
      },
      "payeeReceiveAmount": {
        "amount": "5",
        "currency": "SGD"
      },
      "fees": {
        "amount": "0.05",
        "currency": "SGD"
      },
      "payer": {",
        "partyIdType": "THIRD_PARTY_LINK",
        "partyIdentifier": "payer@test",
        "fspId": "dfspa"
      },
      "payee": {
        "partyIdInfo": {
          "partyIdType": "MSISDN",
          "partyIdentifier": "987654321",
          "fspId": "dfspb"
        }
      },
      "transactionType": {
        "scenario": "TRANSFER",
        "initiator": "PAYER",
        "initiatorType": "CONSUMER"
      },",
      "challenge": "OWZhYjAxZTcwYjU4YzRhMzRmOWQwNzBmZjllZDFiNjc2NWVhMzA1NGI1MWZjZThjZGFjNDEyZDBmNmM2MWFhMQ"
    },
    "currentState": "authorizationReceived"
    "}
  }
}
```
Core connector then takes thirdparty initiation response to respond to PISP's initial request.

```
POST /v3/getTransferFundsQuotationRequest Response
{
  "responseHeader": {
    "responseTimestamp": {
      "epochMillis": new Date.now()
    }
  },
  "result": {
    "success": {
      "feeAmount": {
        "amountMicros": convert_to_amount_micros(initiateResponse.authorization.fees.amount),
        "currencyCode": initiateResponse.body.authorization.fees.currency
      },
      // Assuming this is to support multiple authentication methods.
      // Running with the assumption that Mojaloop will provide a singular FIDO assertion.
      "challengeOptions": [
        {
          // challengeOptionId is used in the GSP transferFunds call.
          // Assuming to identify which authentication method was chosen.
          // We can have the GSP connector generate this
          "challengeOptionId": "xxxxxxxxxxxxxxx" || uuid(),
          "fido": {
            "challenge": initiateResponse.body.authorization.challenge,
            "allowCredentials": [
              // Assuming this is if google wants to only allow specific devices
              // to sign this challenge since they have multi-device support.
              // Mojaloop has no such requirement so we can probably put a mock string here.
              {
                "type": "public-key",
                "id": "xxxxxxxxxxxxxxxxx"
              }
            ]
          }
        }
      ]
    }
  }
}
```

PISP sends transfer request along with user signed challenge to core connector.

```
POST /v3/transferFundsRequest
{
  "requestHeader": {
    "protocolVersion": {
      "major": 3
    },
    "requestId": "bWVyY2hhbnQgdHJhbnN",
    "requestTimestamp": {
      "epochMillis": "1502220196077"
    },
    "paymentIntegratorAccountId": "Mojaloop"
  },
  "googlePaymentToken": {
    "issuerId": {
      "value": "Mojaloop"
    },
    "token": "payer@testbank"
  },
  "transactionDescription": "Payment",
  "amount": {
    "amountMicros": "5050000",
    "currencyCode": "SGD"
  },
  "payeeProxy": {
    "payeeProxyKey": {
      "networkId": "paynow",
      "phoneNumber": "+6577778888"
    },
    "payeeProxyLookupRequestId": "bWVyY2hhbnQgdHJhbnN"
  },
  "getTransferFundsQuotationRequestId": "bWVyY2hhbnQgdHJhbnN",
  "challengeResults": [
    {
      "challengeOptionId": "8StMrYBbh",
      "fidoAssertion": {
        "rawId": "Aad50Szy7ZFb8f7wd",
        "id": "Aad50Szy7ZFb8f7wdfMmFO",
        "response": {
          "authenticatorData": "zHUM-fXe8fPTc7IQdAU8xhonRmZ",
          "signature": "MEUCIHxzf1KZNJTb831gqw0oit-6ms8Do",
          "userHandle": "Kosv9fPtkDoh4Oz7Yq_pVgWHS8H",
          "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uZ2V"
        },
        "type": "public-key"
      }
    }
  ]
}
```

Core connector maps PISP transfer request to thirdparty approve request.

```
POST /thirdpartyTransactions/{transactionRequestId}/approve

{
  authorizationResponse: {
      responseType: 'ACCEPTED',
      signedPayload: {
        signedPayloadType: 'FIDO',
        fidoSignedPayload: {
          id: TransferFundsRequest.challengeResults[0].fidoAssertion.id,
          rawId: TransferFundsRequest.challengeResults[0].fidoAssertion.rawId,
          response: {
            authenticatorData: TransferFundsRequest.challengeResults[0].fidoAssertion.response.authenticatorData,
            clientDataJSON: TransferFundsRequest.challengeResults[0].fidoAssertion.response.clientDataJSON,
            signature: TransferFundsRequest.challengeResults[0].fidoAssertion.response.signature,
            // What is userHandle in FIDO? Ignore for demo.
          },
          type: 'public-key'
        }
      }
    }
};

Response
{
  "transactionStatus": {,
    "transactionRequestState": "ACCEPTED",
    "transactionState": "COMPLETED",
  },",
  "currentState": "transactionStatusReceived",
}
```

Core connector takes approve response and responds to PISP's transferFunds request.
```
POST /v3/transferFundsRequest Response
{
  "responseHeader": {
    "responseTimestamp": {
      "epochMillis": Date.now()
    }
  },
  "result": approveResponse.transactionRequestState == "ACCEPTED" && approveResponse.transactionState == "COMPLETED" ? {"success": {}} : {"creditStatusUnknown": {}},
  "paymentIntegratorTransactionId": "approveResponse.transactionStatus.transactionRequestId"
}
```

## NOTES

Micros Monetary values in this API are represented using a format called "micros", a standard at Google. Micros are an integer based, fixed precision format. To represent a monetary value in micros, multiply the standard currency value by 1,000,000.

For example:

USD$1.23 = 1230000 micro USD

USD$0.01 = 10000 micro USD
