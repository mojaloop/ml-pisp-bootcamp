# GSP API to Thirdparty API mapping

## Points that need attention

1. Quotations in Mojaloop needs the Payer's thirdparty link identifier that is established in the Linking phase in Mojaloop.
   From observation there is no Payer identifying field in `getTransferFundsQuotationRequest`. Mojaloop would need
   some identifier given at this step. In the below mapping example we used `googlePaymentToken` as an interim identifier.
2. Mojaloop uses a transactionRequestId to associate it's three phases of thirdparty initiated payments, that is discovery, agreement and transfer.
   That transactionRequestId is needed from the discovery phase i.e party lookup.
   It is not clear what id we would use to map from the GSP API to transactionRequestId.
   Suitable picks from given examples are `requestHeader.requestId` and `associationId`.

## NOTES

Micros Monetary values in this API are represented using a format called "micros", a standard at Google. Micros are an integer based, fixed precision format. To represent a monetary value in micros, multiply the standard currency value by 1,000,000.

For example:

USD$1.23 = 1230000 micro USD

USD$0.01 = 10000 micro USD

### Mapping example

PISP issues quotation request.
```
POST /v3/getTransferFundsQuotationRequest

const getTransferFundsQuotationRequest = {
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
const transactionRequestId = toUUID(getTransferFundsQuotationRequest.associationId);

POST /thirdpartyTransaction/partyLookup
{
  transactionRequestId: transactionRequestId,
  payee: {
    partyIdType: "MSISDN",
    partyIdentifier: "+6577778888"
  }
}

Response

const partyLookupResponse = {
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
const transactionRequestId = toUUID(getTransferFundsQuotationRequest.associationId);

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

const initiateResponse = {
  body: {
    "authorization": {
      "transactionRequestId": transactionRequestId,
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
const transactionRequestId = initiateResponse.body.authorization.transactionRequestId;

POST /v3/getTransferFundsQuotationRequest Response
const getTransferFundsQuotationResponse = {
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
          // challengeOptionId is used in the GSP transferFunds call, and is the link between the initiated Transfer and the Authorization.
          "challengeOptionId": transactionRequestId,
          "fido": {
            "challenge": initiateResponse.body.authorization.challenge,
            "allowCredentials": [
              // Assuming this is if Google wants to only allow specific devices
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
const transactionRequestId = getTransferFundsQuotationResponse.result.success.challengeOptions[0].challengeOptionId

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
      "challengeOptionId": transactionRequestId,
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
const transactionRequestId = transferFundsRequest.challengeResults[0].challengeOptionId

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
            userHandle: TransferFundsRequest.challengeResults[0].fidoAssertion.response.userHandle,
          },
          type: 'public-key'
        }
      }
    }
};

Response
const approveResponse = {
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
