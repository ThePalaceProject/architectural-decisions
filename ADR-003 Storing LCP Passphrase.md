# ADR-003 Storing LCP Passphrase for Offline Reading

| Property        | Value        |
| --              | --           |
| *Status*        | Active       |
| *Components*    | iOS, Android |
| *Related*       |              |
| *Superceded by* |              |

## Context

*What is the issue that we're seeing that is motivating this decision or change?*

- To support reading/listening of LCP-protected content, a passphrase must be available to enable decryption of the content.
- Publishers have concerns about the potential to permanently decrypt their content and, thus, about the availability of either
    - unprotected content or
    - protected content along with the associated passphrase.

## Decision

### Android

In the Android application, we chose to store the hashed passphrase for the
content in the application-private book database. The passphrase will
be deleted along with the book whenever the application discovers that the
loan for the book has expired.

We decided against any stronger form of protection as we discovered that the
underlying Readium 2 library used to read EPUB files will cache passphrases
indefinitely in its own private database. Additionally, whilst users would
require root access to their devices in order to extract passphrases from
the application's storage, sufficiently competent authenticated users could
also simply view the OPDS feed containing the hashed passphrases using standard
tools such as `curl`. Discussion with the Readium 2 team indicated that the
passphrase is not considered particularly sensitive information:

> Laurent Le Meur:
>
> As far as validation goes, storing the hash of the passphrase in the client
> db is ok. If somebody gets it, the only risk is that this license will be
> overshared and then revoked.

> Hadrien Gardeur:
>
> Holding onto the passphrase is a best practice among LCP based apps as the
> goal is to make things easier for the user.

### iOS

In the iOS application, passphrases are managed by the ReadiumLCP framework. When Readium 2 opens an encrypted asset and tries to read the license file, the ReadiumLCP framework iterates over the available passphrases to decrypt the license. When it cannot find a matching passphrase, the application tries to get a passphrase from the hint in the license file and, in case the hint contains a passphrase, returns the passphrase to the authentication service. This way passphrase is stored in the ReadiumLCP local database only.

ReadiumLCP framework code analysis shows that it stores passphrases indefinitely and never deletes them.

## Alternatives Considered

An alternative to storing the passphrase is to avoid storing it, and to
retrieve it each time the book is opened for reading. This was rejected
for the following reasons:

- It causes the protection to "fail closed": If the network is unavailable,
  the user can't read the book.

- Readium 2 will indefinitely cache the passphrase on first use anyway,
  meaning that any user attempting to locate the passphrase could simply
  look in Readium 2's internal storage and ignore any protection the host
  application was attempting to add.

## Consequences

*What becomes easier or more difficult to do because of this change?*

- Books can be decrypted during reading automatically without the user ever
  even realizing the books are encrypted and protected with a passphrase.
- Books can be read even if the network is unavailable.
