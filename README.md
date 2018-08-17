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

The CMP is a tool that aims to make information transit between the application and the different third parties involved. It's actually the "standard" (IAB way) to share information concerning user consents with other advertising (tracking and so forth) companies.

### Required API

To make communication easier, a contract is etablished between the provider (the user in OUR app) and the consumer (the third party company tool).

The CMP should follow this API:

`__cmp(Command, Parameter, Callback)`

And provide the following as `Command`:

<table>
  <tr>
    <td>Command</td>
    <td>Parameter</td>
    <td>Callback</td>
    <td>Comments</td>
  </tr>
  <tr>
    <td>getVendorConsents</td>
    <td>Vendors Ids (array of string)</td>
    <td>Callback(a VendorConsents object, a success boolean)</td>
    <td>It allows to retrieve information from the consent string concerning the different accepted / denied consents of the current user</td>
  </tr>
  <tr>
    <td>getConsentData</td>
    <td>Consent string [String]</td>
    <td>Callback(VendorConsentData object, success: boolean)</td>
    <td>It provides the consent data of a user for a given string</td>
  </tr>
  <tr>
    <td>ping</td>
    <td> - </td>
    <td>Callback(PingReturn object, success: boolean)</td>
    <td>It only exist to check if the CMP is loaded (not sure about it)</td>
  </tr>
</table>

### Communication with vendors

For communication with vendors we have to rely on our `__cmp` implementation. The communication can be etablished in 3 main ways:

#### `window.__cmp`

The function \_\_cmp is created like this :

```js
export const __cmp = {
  getVendorConsents: () => /* some process to handle the vendor consents */,
  getConsentData: : () => /* some process to handle the vendor consent data */,
  ping: () => /* some process to know if the CMP is loaded. Generally, it's only checking for the iframe locator to exist */,
};
```

Since `__cmp` is bound to window, it'as available everywhere in the app using `window.__cmp`.

#### Via (standard) iframe

The (only for now?) other solution is to rely on iframe communication. The idea is to expose information the a child frame that will expose the information to another of its own child frame (etc... until the last vendor concerned is reached).

We can deal with `iframe` communication using listeners on `message`:

```js
window.addEventListener("message", handleMessage); // makes the communication possible between the app and the third parties
```

:warning: Don't forget to register the `__cmpLocator` iframe to provide the third party information of the loading state of the CMP! (mandatory for the `ping` function):

```js
const iframe = document.createElement("iframe");
iframe.name = "__cmpLocator";
iframe.width = 0;
iframe.height = 0;
iframe.style.display = "none";
document.body.appendChild(iframe);
```

#### Via (safe) iframe

For now, it doesn't seem possible to rely on safeframe. We manage to [open an issue](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/issues/38) on IAB github account, but we didn't get any answer.

And since the documentation is not really precise (seriously, you got me \o/):

> An **updated** safeFrame implementation/specification **will provide** the method $sf.ext.cmp(command, parameter)

We have to read under the line right?
