https://www.6play.fr/parametres-cookies

# IAB Consent Framework

## What is it all about?

### Vendors

In IAB context, `vendors` are all the third parties that need to get consent validations from the current user of our application.
For example, it can be Google Analytics that needs to get information about the user navigation.

They are generally represented as `<script>`, `<iframe>` or even a tracking pixels inside applications.

### Purposes

A purpose is like categorizing `vendors` by their aim, their purpose. For example, it can exist vendors that deal with `Information storage and access` or `Personalisation`.
It simply an object that gather vendors by what they do.

### Features

Feature is something that we don't use at M6Web for now. It seems that they are a particular feature for a particular vendor. If a vendor can have multiple feature dealing with the same purpose, it can
be subcategorized.

### The VendorList

Dealing with IAB, the `VendorList` is an `Object` (weirdo) that aims to describe the different purposes and features per vendors.

It"s composed this way:

```json
{
  "vendorListVersion": 84,
  "lastUpdated": "Some isodate",
  "purposes": [
    {
      "id": 1,
      "name": "Purpose name",
      "description": "Purpose description"
    }
  ],
  "features": [
    {
      "id": "1",
      "name": "Feature name",
      "description": "Feature description"
    }
  ],
  "vendors": [
    {
      "id": "278",
      "name": "Vendor name",
      "policyUrl": "Vendor policy url",
      "purposeIds": ["some", "purpose", "related", "ids"],
      "legIntPurposeIds": ["dont", "know", "the", "aim", "of", "this"],
      "featureIds": ["some", "feature", "ids"]
    }
  ]
}
```

Here the relationships are the following:

- Vendors can relate to multiple purposes
- Vendors can own multiple features
- A feature can concern multiple vendors
- A purpose can contain multiple vendors
- There is _no direct_ relationship between features and purposes.

## The ConsentString

The `Consent String` is a Base64 string generated from information based on the previously described `VendorList` and the different consents provided by the user.
IAB has provided an sdk available on npm & github: https://github.com/InteractiveAdvertisingBureau/Consent-String-SDK-JS

We can use it this way:

```javascript
import { ConsentString } from "consent-string";

const consents = new ConsentString();

/**
 * we have to register this and be sure that our vendor list is at a correct format
 */
consents.setGlobalVendorList(ourGlobalVendorList);

/**
 * Giving current user consent to a specific vendor or purpose
 */
consents.setPurposeAllowed(1, true);
consents.setVendorAllowed(287, true);
```

The `ConsentString` aims to be shared with the vendors that try to get user information. When these vendors will "ask" (see CMP part) for consents, this `ConsentString` will be shared with them. If they are concerned, they can collect information. Else, they can't.

### Vendor based / Purpose based / Feature based

It's possible to manage granularity for consent management. Some website prefers to let the user chose which `vendor` are authorized to gather information.
Some other, like us at M6Web, prefers to use a less granular one and to manage consents at a purpose level.

**Example of a vendor based consent management:**

- [x] Google Analytics
- [ ] Krux

**Example of a vendor based consent management:**

- [ ] Information storage and access
- [x] Personalisation

_Feature based is not available for now with IAB Consent Framework. But since the feature field is available in the VendorList, we can imagine managing consents using feature based approach_

## Consent Management Provider (CMP)
CMP is a company who can read the vendors chosen by a customer (ex: google, analytics ... ) and the company can see the consent status of this user.

### Required API

Each CMP must provide the following API:

`__cmp(Command, Parameter, Callback)`

This API provide few command : 

<table>
  <tr>
    <td>Command</td>
    <td>Parameter</td>
    <td>Callback</td>
    <td>Comments</td>
  </tr>
  <tr>
    <td>getVendorConsents</td>
    <td>Vendors Ids [Array:String]</td>
    <td>Callback(VendorConsents object, success: boolean)</td>
    <td>Vendor consents strings</td>
  </tr>
  <tr>
    <td>getConsentData</td>
    <td>Consent string [String]</td>
    <td>Callback(VendorConsentData object, success: boolean)</td>
    <td>Consents data from user</td>
  </tr>
  <tr>
    <td>ping</td>
    <td> - </td>
    <td>Callback(PingReturn object, success: boolean)</td>
    <td>This function invoke the callback immediatly with some informations about the CMP script has loaded for all user.</td>
  </tr>
</table>

### Communication with vendors

For communication with vendors we mostly use the `__cmp` function for local communication and via `iframe` (standard).

#### `window.__cmp`
The function __cmp is created like this : 
```js
export const __cmp = {
  getVendorConsents: (thirdPartyIds = []) => {
    return new VendorConsents(
      thirdPartyIds,
      [''], // Purpose array
      'Constent string',
    );
  },
  getConsentData: (metadata, gdprApplies = true) => {
    this.consentData = metadata;
    this.gdprApplies = gdprApplies;
    this.hasGlobalScope = false;
  },
  ping: () => {
    const isCmpLoaded = !!document.querySelector('iframe[name=__cmpLocator]');

    return new PingReturn(isCmpLoaded);
  },
};
```

So you can call this function anywhere in the code.

For exemple, you cann get vendor consents :

```js
__cmp.getVendorConsents()
```

#### Via (standard) iframe

Standard iframe is call with eventListener in this function : 
```js
export const bootIabStandardIframe = () => {
  window.addEventListener('message', handleMessage);

  const iframe = document.createElement('iframe');
  iframe.name = '__cmpLocator';
  iframe.width = 0;
  iframe.height = 0;
  iframe.style.display = 'none';
  document.body.appendChild(iframe);
};
```

The iframe always exist since the user has given access to its data. 
if user give consent the app is able to converse with it's child frame.
After that the child frames will send messages to your app and return information concerning the previously agreed consents.

#### Via (safe) iframe
Safe frame is not available but you can see this issue : 

[InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework#38](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/issues/38)

and this code :

[https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/CMP%20JS%20API%20v1.1%20Final.md#via-safeframes-](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/CMP%20JS%20API%20v1.1%20Final.md#via-safeframes-)

> An updated safeFrame implementation/specification  **will provide**  the method $sf.ext.cmp(command, parameter)