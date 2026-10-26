# ForgeRock IDM LDAP-to-Managed-User Reconciliation Investigation

**Report date:** October 20, 2026  
**Product:** ForgeRock IDM 7.4.0  
**Outcome:** The user reported that reconciliation is working after the changes described below.  
**Report location:** Current CLI project root, `c:\Users\mssriram\Documents\forgerock_poc`.

## 1. Scope, evidence, and verification boundaries

This report records the investigation conducted in this conversation: the errors, configuration reads, diagnostic commands, script changes, UI configuration, unsuccessful approaches, and persistence concerns.

Evidence came from:

- User-supplied reconciliation logs.
- User-supplied `onUpdate-user-custom.js` source.
- An attached `check-user-object.js` source snapshot.
- User-supplied `grep` and `sed` output from the IDM container.
- Read-only HTTP requests to the IDM configuration, policy, reconciliation, LDAP connector, and managed-user endpoints.
- Official ForgeRock/Ping Identity documentation for IDM 7.4.
- The user's final confirmation: "it is working now."

Important distinctions:

- The assistant did not modify remote IDM configuration, execute remote script edits, trigger reconciliation, or update user records. The user applied changes through the container and UI. The assistant performed read-only remote checks and created this local report.
- The final mapping-level normalization and username transformation were retrieved from the live configuration and are reproduced exactly below.
- The password-policy guard was visible in part in the user's later script output. The final complete deployed `policy.js` function was not retrieved.
- The object-compatible changes recommended for the managed-user update hook and password-policy history reader are documented exactly as proposed. Their final deployed file contents were not independently retrieved.
- No before-and-after export of the complete `sync.json` was available. The report does not invent unknown original transformation code or claim that unrelated configuration changed.
- Successful reconciliation does not establish restart durability, cluster-wide consistency, or full password-policy regression coverage.
- Credentials, password values, hashes, and actual history entries are deliberately excluded. Authentication examples use placeholders.

## 2. Environment and resources

The supplied IDM endpoint was:

```text
http://idm-clops01-fr.apps.ocpsandbox.ocpnp.calheers.net/openidm
```

Container paths and identifiers observed during the investigation:

```text
/opt/openidm/conf/sync.json
/opt/openidm/conf/system.properties
/opt/openidm/bin/defaults/script/policy.js
/opt/openidm/bin/defaults/script/policyFilter.js
/opt/openidm/script/onUpdate-user-custom.js
/opt/openidm/script/check-user-object.js
```

One shell prompt identified the pod as `forgeops-idm-ss-0`. The difference between that pod name and the route hostname initially raised an environment-correlation question. A subsequent read found the exact failing reconciliation ID on the supplied endpoint, establishing that the logged reconciliation was known there.

### 2.1 Mappings observed

Early configuration inspection returned two inbound mappings:

| Mapping | Source | Target |
|---|---|---|
| `systemOpendjaccount__ACCOUNT___managedUser` | `system/OpenDJAccount/__ACCOUNT__` | `managed/user` |
| `systemOpendjaccountAccount_managedUser` | `system/OpenDJAccount/account` | `managed/user` |

The later error logs identified the second mapping as the failing mapping:

```text
systemOpendjaccountAccount_managedUser
```

Both initially observed mappings contained these property mappings:

| Source | Target | Initial metadata observed |
|---|---|---|
| `uid` | `userName` | Transform present; no condition or default |
| `vEmail` | `vEmail` | Transform and condition present; no default |
| `vMobile` | `vMobile` | Transform and condition present; no default |

The user initially referred to a phone attribute as `vPhone`; the actual configuration used `vMobile`.

Neither `password` nor `fieldHistory` was mapped from LDAP.

The final read returned the `systemOpendjaccountAccount_managedUser` configuration documented in this report. It did not return the other initially observed mapping. This investigation did not establish whether that other mapping was removed, renamed, or otherwise changed.

### 2.2 Situation policy

The relevant mapping policy remained:

```json
{
  "action": "UPDATE",
  "situation": "CONFIRMED"
}
```

No change to this action was recommended or verified.

## 3. Initial problem and interpretation

The initial reconciliation log contained:

```text
org.forgerock.openidm.sync.SynchronizationException:
Policy validation failed
```

The significant nested details were:

```text
code=403
result=false
password:
  policyRequirement=Cannot use the same password
fieldHistory:
  policyRequirement=VALID_TYPE
  invalidType=string
  validTypes=[object]
```

Relevant stack frames included:

```text
org.forgerock.openidm.sync.SyncOperation.updateTargetObject
org.forgerock.openidm.sync.SourceRecon.recon
org.forgerock.openidm.router.filter.ScriptedFilter.filterUpdate
/opt/openidm/bin/defaults/script/policyFilter.js#68
```

The first supplied reconciliation identifier was:

```text
e2403525-dd7c-434f-8f21-c13a00a6698a-1135270
```

The log ended with a reconciliation summary containing `CONFIRMED: 1`, even though the update had failed.

### 3.1 Why CONFIRMED did not mean success

Official documentation distinguishes the synchronization situation from the action performed afterward. The observed sequence was:

```text
Source and target association assessed
    -> CONFIRMED
    -> configured UPDATE action
    -> managed-user policy validation
    -> update rejected
```

`CONFIRMED` was not proof that the target update succeeded. Likewise, a "Reconciliation completed" message was not sufficient evidence that every object update succeeded.

The evidence pointed to rejection of the managed-user update through IDM's policy filter, not a failure to locate or associate the LDAP account.

## 4. Documentation-based triage

The investigation used official documentation rather than treating product behavior recalled from training as authoritative.

Initial discovery included current PingIDM documentation. Once the user supplied version `7.4.0`, the investigation switched to the available IDM 7.4 pages.

The relevant documented behavior was:

- Situations are assessed first; the configured mapping policy then chooses an action.
- Objects and properties are automatically validated on creation, update, and patch.
- Password-history enforcement uses an `is-new` policy.
- The documented `fieldHistory` schema type is `object`.
- `fieldHistory` is a map from field names to lists of historical values.
- The documented sample does not evaluate an unchanged current password.
- Mapping `onUpdate` scripts have access to `target` and `oldTarget`.
- A selected property mapping supplies the selected source value to its transform.
- Script resolution uses configured source directories in reverse order, allowing project overrides of bundled scripts.
- Configuration persistence and writing configuration JSON files are distinct concerns.

Some guessed documentation URLs returned 404. Those failed fetches were not used as evidence. Search-result summaries were used to discover pages; the relevant official pages were then fetched directly.

## 5. Connectivity and authentication diagnostics

### 5.1 DNS and HTTPS

The supplied hostname resolved successfully to private network addresses.

The first authenticated schema request was attempted over HTTPS. It failed during certificate verification:

```text
curl: (60) schannel: SEC_E_UNTRUSTED_ROOT
The certificate chain was issued by an authority that is not trusted.
```

TLS verification was not disabled.

### 5.2 HTTP route

The user instructed use of HTTP. An unauthenticated HTTP probe returned:

```text
HTTP/1.1 401 Unauthorized
```

It did not redirect to HTTPS. Authorized read-only requests then used the supplied HTTP endpoint and IDM authentication headers.

Security implications were explicitly raised:

- HTTP transmits authentication headers without TLS protection.
- A trusted HTTPS route is preferable.
- The credential shared in chat should be rotated.
- No credentials are included in this report.

The assistant used a curl configuration supplied through standard input for authentication rather than reproducing credentials in report commands. Responses containing user data were reduced to the necessary identifiers, attribute names, or type metadata before output.

## 6. Live configuration and policy inspection

### 6.1 Endpoints read

The investigation used these read-only endpoints:

```text
GET /openidm/config/managed
GET /openidm/config/policy
GET /openidm/config/sync
GET /openidm/policy/managed/user
```

The effective policy response confirmed:

```text
password:
  policy requirements: VALID_TYPE, IS_NEW
  valid type: string
  policyId: is-new
  historyLength: 24

fieldHistory:
  policy requirement: VALID_TYPE
  valid type: object
```

### 6.2 Final verified managed-user configuration

Relevant password schema settings were:

```json
{
  "type": "string",
  "scope": "private",
  "title": "Password",
  "policies": [
    {
      "params": {
        "historyLength": 24
      },
      "policyId": "is-new"
    }
  ],
  "viewable": false,
  "isVirtual": false,
  "encryption": {
    "purpose": "idm.password.encryption"
  },
  "isPersonal": false,
  "searchable": false,
  "description": "Password",
  "isProtected": true,
  "userEditable": true,
  "usageDescription": null,
  "deleteQueryConfig": false
}
```

The verified `fieldHistory` schema was:

```json
{
  "type": "object",
  "scope": "private",
  "title": "Field History",
  "viewable": false,
  "searchable": false,
  "userEditable": false
}
```

The final lifecycle-hook configuration was:

```json
{
  "onCreate": {
    "file": "script/check-user-object.js",
    "type": "text/javascript",
    "globals": {
      "historySize": 24,
      "historyFields": [
        "password"
      ]
    }
  },
  "onUpdate": {
    "file": "script/onUpdate-user-custom.js",
    "type": "text/javascript",
    "globals": {
      "historySize": 24,
      "historyFields": [
        "password"
      ]
    }
  }
}
```

An earlier diagnostic projection looked for `historyFields` and `historySize` directly on the hook object and printed null. The final read shows they are configured under `globals`. The earlier null output was not evidence that these parameters were absent.

Other hook references observed during the investigation included:

```text
postCreate -> script/set-password-status.js
postDelete -> inline script
```

Their code was not part of the changes established here.

### 6.3 Policy script registration

The policy configuration contained:

```json
{
  "file": "policy.js",
  "additionalFiles": []
}
```

The existence of a separate `pwpolicy.js` file therefore did not establish that it was loaded through `additionalFiles`.

## 7. Script discovery and source inspection

The user ran the following commands inside the Linux IDM container.

### 7.1 Locate the rejection and policy registration

```bash
grep -rnlF --include='*.js' \
  'Cannot use the same password' /opt/openidm

grep -rnF --include='*.js' \
  'is-new' /opt/openidm

grep -rnF --include='*.js' \
  'IS_NEW' /opt/openidm
```

The reported `is-new` matches were:

```text
/opt/openidm/bin/defaults/script/policy.js:187
/opt/openidm/samples/multiple-passwords/script/pwpolicy.js:12
/opt/openidm/script/pwpolicy.js:12
```

Additional locating commands supplied were:

```bash
find /opt/openidm -type f \
  \( -name 'policy.js' -o -name 'pwpolicy.js' -o -name 'script.json' \) \
  -print

grep -nF -A 15 'policyFunctions.isNew' \
  /opt/openidm/bin/defaults/script/policy.js
```

### 7.2 Inspect the active policy definition

```bash
grep -nF -B 20 -A 100 \
  '"policyId" : "is-new"' /opt/openidm/bin/defaults/script/policy.js

grep -nF -B 20 -A 50 \
  'Cannot use the same password' \
  /opt/openidm/bin/defaults/script/policy.js \
  /opt/openidm/script/pwpolicy.js \
  /opt/openidm/samples/multiple-passwords/script/pwpolicy.js

grep -nE 'pwpolicy|require[[:space:]]*\(|load[[:space:]]*\(' \
  /opt/openidm/bin/defaults/script/policy.js
```

The registration in `policy.js` was:

```javascript
{
    "policyId" : "is-new",
    "policyExec" : "isNew",
    "policyRequirements" : ["IS_NEW"],
    "clientValidation": false,
    "validateOnlyIfPresent": true
}
```

`validateOnlyIfPresent` was not a "validate only if changed" setting.

The import search showed:

```text
var _ = require('lib/lodash')
additionalPolicyLoader.load(additionalPolicies)
```

It did not show a direct import of `pwpolicy.js`.

### 7.3 Search all deployed history references

```bash
grep -rnF --include='*.js' -B 4 -A 8 \
  'fieldHistory' \
  /opt/openidm/script \
  /opt/openidm/bin/defaults/script
```

Relevant findings:

| File | Observed behavior |
|---|---|
| `check-user-object.js` | Initializes `object.fieldHistory = {}` and adds per-field arrays |
| `onCreate-user-custom.js` | Also initializes an object-shaped history |
| `onCreate-fpactUser-custom.js` | Initializes object-shaped history for a different managed type |
| `onUpdate-user-custom.js` | Unconditionally parses `oldObject.fieldHistory` as JSON text |
| `onUpdate-fpactUser-custom.js` | Contains a similar unconditional parse for a different managed type |
| `onUpdate-user.js` | Contains legacy array-based history handling and copies old history |
| `pwpolicy.js` | Contains another `isNew` implementation and unchanged-password rejection |
| `validatePassword.js` | Parses `fieldHistory` as text while reading user password history |
| `validatePassword-fpactUser.js` | Has similar parsing for a different managed type |
| `bin/defaults/script/policy.js` | Contains the registered custom `isNew` implementation |

The legacy `onUpdate-user.js` source included logging of encrypted/decrypted values. It was not established as the active managed-user hook and was not changed in this investigation. No such values are reproduced here.

The search did not conclusively locate an assignment that serialized `fieldHistory` into a string. Object-versus-string inconsistency was established, but the original producer of the string was not.

### 7.4 Inspect the policy filter

```bash
sed -n '1,100p' /opt/openidm/bin/defaults/script/policyFilter.js
```

The supplied filter source included:

```javascript
namespace.evaluatePolicy = function(path, content) {
    return openidm.action(
        "policy/" + path,
        "validateObject",
        content,
        { "external" : "true" }
    );
};
```

Its request processing used:

```javascript
result = this.evaluatePolicy(fullResourcePath, request.content);

if (!result.result) {
    throw {
        "code" : 403,
        "message" : "Policy validation failed",
        "detail" : result
    };
}
```

This established that changing a local `fieldHistory` variable inside `isNew` does not normalize the update request being checked by the separate schema policy.

`policyFilter.js` was inspected only. No change to the filter or policy enforcement was recommended.

## 8. Change 1: Skip history validation for an unchanged password

**File:** `/opt/openidm/bin/defaults/script/policy.js`  
**Function:** `policyFunctions.isNew(fullObject, value, params, property)`

`property` is a function parameter identifying the validated property; in this case it is `password`. It is not a new global variable that needs to be declared.

### 8.1 Original rejection

The original user-supplied source around lines 693-697 contained:

```javascript
// Don't enforce this policy if the password hasn't changed
if (currentObject[property] === value) {
//                logger.info("Same Password Field :: " +currentObject[property].length);
    //return [];
    return [{"policyRequirement": "Cannot use the same password"}];
}
```

The implementation contradicted its comment: it rejected equality rather than accepting an unchanged password.

The exact error string matched the log, establishing that the failed validation reached this equality branch. The profile-only mapping did not need to map a password to encounter this check.

### 8.2 Recommended insertion

After retrieving and checking `currentObject`, but before parsing or evaluating history, insert:

```javascript
var currentValue = currentObject[property];
var incomingValue = value;

if (openidm.isEncrypted(currentValue)) {
    currentValue = openidm.decrypt(currentValue);
}

if (openidm.isEncrypted(incomingValue)) {
    incomingValue = openidm.decrypt(incomingValue);
}

if (currentValue === incomingValue) {
    return [];
}
```

Remove or disable the original equality-rejection block shown above.

The existing historical-password comparison loop was to remain intact:

```javascript
for (i = numOfFields - 1;
        i >= (numOfFields - historyLength) && i >= 0;
        i--) {
    if ((openidm.isHashed(lastFieldValues[i]) &&
            openidm.matches(value, lastFieldValues[i])) ||
            (lastFieldValues[i] === value)) {
        return [{"policyRequirement": "IS_NEW"}];
    }
}
```

### 8.3 Evidence after the change

Later user-supplied grep output showed a `return []` before the history parse and the old rejection commented out. Subsequent logs no longer listed `Cannot use the same password` as a failed requirement.

The complete final deployed function was not retrieved; this report reproduces the exact recommended insertion, not an invented whole-file replacement.

### 8.4 Behavior and limitation

- An unchanged decrypted password bypasses this history policy.
- A different password proceeds to the existing history checks.
- An explicit submission of the current password also passes the equality guard.
- Distinguishing an explicit same-password change from a profile update carrying the same value would require additional request-aware behavior. That was discussed but not implemented here.

## 9. Change 2: Convert LDAP uid to a scalar userName

**Configuration:** Mapping `properties` entry with `source: "uid"` and `target: "userName"`  
**Storage view:** `/opt/openidm/conf/sync.json`

A later error added:

```text
property=userName
policyRequirement=VALID_TYPE
invalidType=array
validTypes=[string]
```

The actual LDAP response subsequently confirmed:

```json
{
  "uid": ["HELLO.WORLD"]
}
```

### 9.1 Exact final live transformation

The following script was retrieved from the live mapping at report preparation:

```javascript
(function () {
    var username = source;

    if (Array.isArray(username)) {
        if (username.length !== 1) {
            throw new Error(
                "uid must contain exactly one value for userName mapping"
            );
        }
        username = username[0];
    }

    if (typeof username !== "string" || username.trim().length === 0) {
        throw new Error("uid must be a non-empty string");
    }

    return username;
}());
```

The verified mapping entry had this structure:

```json
{
  "target": "userName",
  "source": "uid",
  "transform": {
    "type": "text/javascript",
    "globals": {},
    "source": "<exact JavaScript shown above>"
  }
}
```

The placeholder above is explanatory; it is not a literal value to deploy.

### 9.2 Intended behavior

| Input | Result |
|---|---|
| `"HELLO.WORLD"` | Same string |
| `["HELLO.WORLD"]` | `"HELLO.WORLD"` |
| Empty array | Explicit error |
| Array with multiple values | Explicit error |
| Missing, null, or non-string scalar | Explicit error |
| Empty or whitespace-only string | Explicit error |

The script validates trimmed length but returns the original string. It does not trim, lowercase, uppercase, or otherwise rename the user.

The original transform existed but its exact source was not captured before replacement.

## 10. Managed-user identity lookup and history inspection

### 10.1 First ID lookup failed

The user initially supplied:

```text
f9e00aa3-6f99-40b0-9a13-fbebdf17fce4
```

A read against:

```text
GET /openidm/managed/user/f9e00aa3-6f99-40b0-9a13-fbebdf17fce4?_fields=fieldHistory
```

returned:

```text
404 Not Found
Object managed/user/f9e00aa3-6f99-40b0-9a13-fbebdf17fce4
not found in managed/user
```

The configuration endpoint returned HTTP 200, distinguishing the missing object from an unavailable IDM service.

### 10.2 LDAP lookup established the ID belonged to the connector

The user supplied a query equivalent to:

```bash
curl --get \
  "$IDM_BASE/system/OpenDJAccount/__ACCOUNT__" \
  --data-urlencode '_queryFilter=uid eq "hello.world"' \
  --header "X-OpenIDM-Username: $IDM_USERNAME" \
  --header "X-OpenIDM-Password: $IDM_PASSWORD"
```

It returned one LDAP account with:

```text
connector _id: f9e00aa3-6f99-40b0-9a13-fbebdf17fce4
uid: ["HELLO.WORLD"]
```

Only the necessary identifier, UID, and attribute names were displayed by the assistant. Password and other account attribute values were not displayed.

### 10.3 Corresponding managed user

A managed-user query used:

```text
userName eq "HELLO.WORLD" or userName eq "hello.world"
```

with requested fields:

```text
_id,userName,fieldHistory
```

It found:

```text
managed-user _id: 0113b621-7691-49cf-bd92-18531eb1f068
userName: HELLO.WORLD
```

A subsequent direct read explicitly requested `_id,fieldHistory`. Its returned property names were only:

```text
_id
_rev
```

Therefore, `fieldHistory` was omitted from that response. The initial diagnostic described it as "null or absent"; the follow-up established omission rather than an explicitly returned null.

This did not establish whether history was absent in storage, omitted during retrieval, or represented differently elsewhere. It did not prove the original string's source.

## 11. Change 3: Make managed-user update history initialization object-compatible

**File:** `/opt/openidm/script/onUpdate-user-custom.js`

### 11.1 Original source

The supplied update hook began with:

```javascript
// Initialize the fieldHistory object
//object.fieldHistory = oldObject.fieldHistory || {};
object.fieldHistory= new Object();
object.fieldHistory =JSON.parse(oldObject.fieldHistory) || {};
```

It then iterated through `historyFields`, initialized per-field arrays, compared old and new values, and shifted/pushed history when it considered a value changed.

The original comparison logic decrypted the old value but not necessarily the new value. Earlier guidance noted that comparisons should use compatible decrypted values. A complete replacement of that loop was not supplied or independently verified.

### 11.2 Exact recommended replacement for initialization

Replace the two active initialization assignments with:

```javascript
var storedHistory = oldObject.fieldHistory;

if (typeof storedHistory === "string") {
    try {
        storedHistory = JSON.parse(storedHistory);
    } catch (error) {
        throw new Error("Stored fieldHistory contains invalid JSON");
    }
}

if (typeof storedHistory === "undefined" || storedHistory === null) {
    storedHistory = {};
}

if (typeof storedHistory !== "object" ||
        Array.isArray(storedHistory)) {
    throw new Error("Stored fieldHistory must be an object");
}

object.fieldHistory = storedHistory;
```

This accepts already-object history and legacy object-shaped JSON strings. It rejects arrays, other scalars, and invalid JSON. It does not replace invalid existing history with an empty success-shaped fallback.

The existing creation behavior initializes history as an object. Keeping this update path compatible avoids introducing an unconditional `JSON.parse` failure once history is correctly object-valued.

### 11.3 Scope and verification

The final deployed contents of this file were not retrieved. This is the exact recommended replacement, not independently verified whole-file content.

The rest of the user-supplied hook contained last-change timestamps, user-type lookup, password expiry/warning calculations, request handling, and nested patches. No edits to those sections were established in this investigation.

## 12. Change 4: Make the policy's history reader object-compatible

**File:** `/opt/openidm/bin/defaults/script/policy.js`  
**Location:** Inside `policyFunctions.isNew`, after the unchanged-password guard.

### 12.1 Original source

```javascript
fieldHistory= new Object();
fieldHistory = JSON.parse(currentObject.fieldHistory);
```

### 12.2 Exact recommended replacement

```javascript
fieldHistory = currentObject.fieldHistory;

if (typeof fieldHistory === "string") {
    try {
        fieldHistory = JSON.parse(fieldHistory);
    } catch (error) {
        throw new Error("Stored fieldHistory contains invalid JSON");
    }
}

if (fieldHistory !== null &&
        typeof fieldHistory !== "undefined" &&
        (typeof fieldHistory !== "object" ||
         Array.isArray(fieldHistory))) {
    throw new Error("Stored fieldHistory must be an object");
}
```

Keep the existing missing-history check and historical-password comparison loop after this block.

### 12.3 Scope and verification

The final deployed replacement was not independently retrieved.

This change makes the policy reader compatible with an object-valued history. It does not modify the managed-user update payload and was not sufficient by itself to fix the separate `VALID_TYPE` failure.

## 13. Why errors persisted during intermediate attempts

Two later reconciliation logs identified:

```text
f2c2afcc-f3be-4bbf-80b9-3ca6f54266ae-107994
f2c2afcc-f3be-4bbf-80b9-3ca6f54266ae-109178
```

Both later logs listed only:

```text
property=fieldHistory
policyRequirement=VALID_TYPE
invalidType=string
validTypes=[object]
```

Neither listed the earlier password-reuse rejection or the `userName` array failure. This was evidence of a changed failure set, not proof of complete success.

The second log contained these application messages before rejection:

```text
validations started ....
Inside policy.js
Inside Is New method 10-12-2021. Updated 13-01-2022
Updated new method
```

It did not supply the newly proposed mapping-normalization markers.

The assistant read the live configuration and found:

```json
{
  "name": "systemOpendjaccountAccount_managedUser",
  "onUpdate": null,
  "propertyTargets": [
    "userName",
    "vEmail",
    "vMobile"
  ]
}
```

This established a concrete deployment/configuration gap: the proposed mapping hook was not currently configured on the active failing mapping.

The reconciliation metadata endpoint for `...-109178` returned that ID, the expected mapping name, and `state: "SUCCESS"`. Because the user-supplied object update log still contained a failure, that metadata state was not treated as proof that the particular user update succeeded.

## 14. Change 5: Normalize target history in the mapping before the update

**Mapping:** `systemOpendjaccountAccount_managedUser`  
**Configuration level:** Top-level mapping `onUpdate`  
**Configuration-file view:** `/opt/openidm/conf/sync.json`

This hook is distinct from:

- The managed-user `onUpdate` lifecycle hook.
- The `uid` property transform.
- The `CONFIRMED` policy condition.
- The `isNew` password-policy function.

### 14.1 Exact final live script

The following script was retrieved from the live configuration during report preparation:

```javascript
var history = target.fieldHistory;

logger.info(
    "Recon history normalization: before=" + typeof history
);

if (typeof history !== "undefined") {
    if (typeof history === "string") {
        try {
            history = JSON.parse(history);
        } catch (error) {
            throw new Error("fieldHistory contains invalid JSON");
        }
    }

    if (history === null ||
            typeof history !== "object" ||
            Array.isArray(history)) {
        throw new Error("fieldHistory must be an object");
    }

    target.fieldHistory = history;
}

logger.info(
    "Recon history normalization: after=" + typeof target.fieldHistory
);

target;
```

The verified script descriptor contained:

```json
{
  "type": "text/javascript",
  "globals": {},
  "source": "<exact JavaScript shown above>"
}
```

The placeholder above is explanatory; it is not a literal value to deploy.

An earlier proposed version wrapped equivalent normalization in an immediately invoked function and returned `target`. The final UI-applied version above is the one verified live.

### 14.2 Behavior

| Target history before the hook | Behavior |
|---|---|
| Undefined | Leave untouched |
| Object | Preserve the object |
| String containing an object-shaped JSON value | Parse and assign the resulting object to `target.fieldHistory` |
| Invalid JSON string | Throw an explicit error |
| Null | Throw an explicit error |
| Array | Throw an explicit error |
| Other scalar | Throw an explicit error |

This hook repairs the update payload at the mapping boundary. It preserves entries rather than resetting history or disabling validation.

It is a targeted compatibility normalization; it does not identify the original producer of the string or prove that every other write path is corrected.

### 14.3 UI steps used

The user was instructed to:

1. Stop any running reconciliation for the mapping.
2. Open **Configure > Mappings** in the IDM 7.4 Admin UI.
3. Open **systemOpendjaccountAccount_managedUser**, in the OpenDJ-to-managed-user direction.
4. Select **Behaviors**.
5. Expand **Situational Event Scripts**.
6. Open the **Update / onUpdate** editor.
7. Select JavaScript.
8. Paste the JavaScript source only, not the JSON descriptor.
9. Save the editor and mapping as applicable.
10. Keep the `CONFIRMED` action as `UPDATE`.

The subsequent read-only configuration check verified that the hook existed and its inline source contained the normalization marker.

### 14.4 Diagnostic markers

The script logs types only:

```text
Recon history normalization: before=string
Recon history normalization: after=object
```

Interpretation:

| Markers | Meaning |
|---|---|
| `string` then `object` | Conversion occurred at the hook |
| `object` then `object` | History already had the expected outer type |
| `undefined` then `undefined` | History was not present on the mapping target |
| No markers | Hook execution or correct log selection is not established |

A log search command supplied was:

```bash
grep -nF 'Recon history normalization:' /opt/openidm/logs/openidm*.log
```

`after=object` does not alone prove successful reconciliation. Later entries must also be checked for policy or update failures.

The user did not supply the final successful run's marker output or full reconciliation log.

## 15. Configuration write failure and persistence analysis

After the UI save, the user reported:

```text
Updating configuration file: file:/opt/openidm/conf/sync.json
Unable to save configuration
java.io.FileNotFoundException:
/opt/openidm/conf/sync.json (Read-only file system)
```

The exception occurred in:

```text
org.forgerock.openidm.config.installer.JSONConfigInstaller.configurationEvent
```

### 15.1 What was verified

A read of the running configuration returned:

```text
onUpdateConfigured: true
scriptType: text/javascript
scriptFile: null
containsNormalizationMarker: true
hasInlineScript: true
```

Therefore:

- The hook was present in the running configuration.
- Writing the file-system view of `sync.json` failed.
- The exception did not prove that the runtime update had been rejected.
- Restart/redeployment persistence remained unverified.

### 15.2 Product behavior

IDM 7.4 documentation states that its repository is normally the authoritative configuration source and that JSON files are maintained as a view. Persistent repository configuration can be disabled using:

```properties
openidm.config.repo.enabled=false
```

With that setting, configuration is memory-only.

The diagnostic command supplied was:

```bash
grep -nE '^[[:space:]]*openidm\.config\.repo\.enabled[[:space:]]*=' \
  /opt/openidm/conf/system.properties
```

The output of that command was not supplied, so the repository-persistence mode was not established.

### 15.3 Durable deployment guidance

The user was advised to:

- Preserve the runtime change before restarting the pod.
- Update the authoritative configuration profile, image source, or mounted configuration that supplies `sync.json`.
- Redeploy using the deployment's normal controlled process.
- Verify the actual source contains the new `onUpdate` script.
- Not assume a file-based export captured a runtime-only change after a file write failed.
- Not attempt to fix a read-only filesystem with `chmod`.

ForgeOps documentation describes a customize/export/profile/build/redeploy workflow. Its exact application depends on the installed ForgeOps version and how this deployment supplies configuration.

No mount changes, image builds, configuration-profile edits, permission changes, persistence-setting changes, or redeployments were performed or independently verified in this investigation.

## 16. Files and configuration changes: final accounting

| File or configuration | Action | Evidence/status |
|---|---|---|
| `bin/defaults/script/policy.js` | Add unchanged-password guard before history access; remove equality rejection | Partial changed source supplied; subsequent logs omitted the rejection |
| `bin/defaults/script/policy.js` | Accept object or JSON-string history in the policy reader | Exact recommendation recorded; final deployed block not retrieved |
| `script/onUpdate-user-custom.js` | Replace unconditional parse with object-compatible history initialization | Exact recommendation recorded; final deployed block not retrieved |
| Active mapping `uid -> userName` transform | Convert singleton array to string; reject invalid/ambiguous values | Exact final live script verified |
| Active mapping top-level `onUpdate` | Normalize target history before managed update | Initially absent; exact final live script verified |
| `conf/sync.json` | Intended file view of mapping changes | File write failed with read-only filesystem error |
| `conf/managed.json` / `config/managed` | Schema and hook metadata inspected | No schema change established; history type remained object |
| `conf/policy.json` / `config/policy` | Policy registration inspected | `policy.js`, empty `additionalFiles`; no registration change established |
| `bin/defaults/script/policyFilter.js` | Inspected validation boundary | Not changed |
| `script/check-user-object.js` | Inspected creation history initialization and other user logic | No change established |
| `script/pwpolicy.js` | Inspected duplicate/custom history policy | Loading not established; no change established |
| Sample `pwpolicy.js` | Search/reference only | No change established |
| Other history-related scripts | Search results reviewed | No changes established |
| This report | Created in the current CLI project root | Persistent local documentation artifact |

No evidence establishes changes to `vEmail` or `vMobile` mapping logic.

## 17. Outcome and remaining limits

### 17.1 Established outcome

The user reported:

```text
it is working now
```

The final read independently confirmed:

- The intended mapping has the singleton-array username transformation.
- The same mapping has the history-normalization `onUpdate` script.
- `CONFIRMED` still invokes `UPDATE`.
- The history schema remains `object`.
- Password policy `is-new` remains configured with history length 24.
- The managed-user history hooks receive `historyFields: ["password"]` and `historySize: 24` through `globals`.

### 17.2 Root-cause conclusions

1. **Password failure:** The custom `isNew` branch rejected an unchanged password. Its error text matched the original log. The guard corrected that behavior for profile updates.
2. **Username failure:** LDAP returned `uid` as an array while `userName` required a string. The verified transform corrected that shape mismatch.
3. **History failure:** Policy validation saw string-valued `fieldHistory` where an object was required. Local parsing in `isNew` did not normalize the update payload. The final mapping hook normalizes that target property before the update.
4. **Intermediate failure to apply the proposed fix:** The mapping's live `onUpdate` was null when checked. The UI-applied hook was subsequently verified.
5. **Persistence problem:** The runtime configuration contained the hook, but writing `sync.json` failed because the filesystem was read-only.

### 17.3 Not established

- The original writer or serialization path that produced string-valued history.
- The complete final deployed contents of the container-edited JavaScript files.
- Which branch the final normalization hook took in the successful run.
- Successful prevention of every historical-password reuse scenario after the edits.
- Correctness of explicit same-password submission behavior against the deployment's business requirements.
- Durable persistence across restart, image replacement, or redeployment.
- Uniform configuration and script contents across all IDM replicas.
- A completed migration of legacy history data.

No policy enforcement was intentionally disabled, no history was intentionally cleared, and no schema relaxation to `string` was recommended.

## 18. Follow-up acceptance checks

These checks remain useful before treating the change as production-complete. They are not represented as already executed:

1. A profile-only reconciliation updates the intended attributes without password-history rejection.
2. A singleton LDAP UID maps to a scalar username; empty and multi-value UIDs fail explicitly.
3. Password-change history checks still reject prohibited historical values.
4. The explicit-current-password behavior is accepted by the deployment's policy owner or handled separately.
5. Existing history entries are preserved after successful updates.
6. Malformed history produces an explicit error rather than being silently replaced.
7. Authoritative deployment configuration contains the verified mapping hook and transform.
8. The configuration remains present after a controlled restart/redeployment.
9. All serving IDM replicas use the intended configuration and script version.
10. Diagnostic type logging is retained or removed according to the operational logging policy; no history values are logged.
11. The shared credential is rotated and a trusted HTTPS route is used for subsequent administrative access.

## 19. Official documentation references

1. [IDM 7.4 - Synchronization situations and actions](https://docs.pingidentity.com/pingidm/7.4/synchronization-guide/chap-situations-actions.html)
   - Situation/action distinction, mapping policies, and Admin UI Behaviors navigation.
2. [IDM 7.4 - Use policies to validate data](https://docs.pingidentity.com/pingidm/7.4/objects-guide/policies.html)
   - Automatic object/property validation on create, update, and patch.
3. [IDM 7.4 - Password history policy](https://docs.pingidentity.com/pingidm/7.4/security-guide/passwords.html#password-history)
   - `is-new`, object-shaped `fieldHistory`, hook parameters, and unchanged-current-password behavior.
4. [IDM 7.4 - Script configuration](https://docs.pingidentity.com/pingidm/7.4/scripting-guide/script-config.html)
   - Script source directories, reverse search order, and recompilation configuration.
5. [IDM 7.4 - Script triggers defined in mappings](https://docs.pingidentity.com/pingidm/7.4/scripting-guide/script-triggers-mappings.html)
   - Mapping `onUpdate`, `target`, `oldTarget`, and transform variables.
6. [IDM 7.4 - Synchronization reference](https://docs.pingidentity.com/pingidm/7.4/synchronization-guide/synchronization-ref.html)
   - Mapping, property, policy, and script object structure.
7. [IDM 7.4 - Resource mapping](https://docs.pingidentity.com/pingidm/7.4/synchronization-guide/mappings.html)
   - Configuration locations and avoiding mapping changes during active reconciliation.
8. [IDM 7.4 - Script triggers defined in managed configuration](https://docs.pingidentity.com/pingidm/7.4/scripting-guide/script-triggers-managedConfig.html)
   - Managed-user hook variables and request context.
9. [IDM 7.4 - Managed objects reference](https://docs.pingidentity.com/pingidm/7.4/objects-guide/appendix-managed-objects.html)
   - Managed-object update semantics and lifecycle hooks.
10. [IDM 7.4 - Configuration changes](https://docs.pingidentity.com/pingidm/7.4/setup-guide/changing-configuration.html)
    - Authoritative repository configuration, JSON-file updates, and memory-only configuration.
11. [ForgeOps 7.5 - IDM image customization](https://docs.pingidentity.com/forgeops/7.5/customize/idm.html)
    - Supporting deployment guidance for configuration profiles, images, and redeployment. This reference does not establish the installed ForgeOps version.

## 20. Evidence inventory

The investigation used these user-provided artifacts:

| Artifact | Contents |
|---|---|
| Initial attached log, 105 lines | Original password/history policy failure and CONFIRMED count |
| Inline `onUpdate-user-custom.js` | Existing history parsing, value comparison, timestamps, and expiry logic |
| Attached `check-user-object.js`, 339 lines | Creation-time user handling and object-shaped history initialization |
| Inline policy grep output | `is-new` registration and exact unchanged-password rejection |
| Session attachment `paste-1792471324093.txt`, 214 lines | Later failure containing only history VALID_TYPE |
| Session attachment `paste-1792471397848.txt`, 313 lines | Deployed script references to `fieldHistory` |
| Inline `policyFilter.js` sed output | Policy evaluation of `request.content` |
| Session attachment `paste-1792471905976.txt`, 213 lines | Repeated history VALID_TYPE failure before live hook verification |
| Inline configuration installer log | Read-only filesystem failure writing `sync.json` |
| Read-only REST observations | Configuration, effective policy, identity lookup, hook presence, and final mapping code |
| Final user confirmation | Reconciliation working after the changes |

The original full Java stack traces repeat framework call chains. Their material exception details, unique reconciliation identifiers, validation boundary, and relevant frames are preserved in this report; repetitive frames and sensitive values are not duplicated.
