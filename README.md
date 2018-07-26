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

https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/CMP%20JS%20API%20v1.1%20Final.md

### Required API

https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/CMP%20JS%20API%20v1.1%20Final.md#what-api-will-need-to-be-provided-by-the-cmp-
https://github.m6web.fr/m6web/site-6play-v4/blob/master/src/app/modules/gdpr/iab/cmp/consentManager.js
https://github.m6web.fr/m6web/site-6play-v4/blob/master/src/app/modules/gdpr/iab/cmp/model.js

### Communication with vendors

https://github.m6web.fr/m6web/site-6play-v4/blob/master/src/app/modules/gdpr/iab/cmp/cmp.js

#### `window.__cmp`

https://github.m6web.fr/m6web/site-6play-v4/blob/master/src/app/modules/gdpr/iab/cmp/cmp.js#L68

#### Via (standard) iframe

https://github.m6web.fr/m6web/site-6play-v4/blob/master/src/app/modules/gdpr/iab/cmp/cmp.js#L33

#### Via (safe) iframe

/!\ Celui la il existe pas ENCORE
