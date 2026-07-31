# Form Validation Message Conventions

Applies to every form's validation schema (Zod, or whatever validation library a project
confirms) regardless of which UI component library renders the message — this is a content/UX
convention, not tied to `ascendra-ui` specifically. Referenced by `/implement-story`'s Web
implementation rules.

---

## Rule 1 — Every validation message names the actual field, never a generic word

A message must say what's wrong **and which field it's about**, in plain language a user would
say out loud. Never leave a message as a bare, generic word or phrase with no field name in it.

| Don't | Do |
|---|---|
| `"Required"` | `"Legal name is required"` |
| `"Invalid email"` | `"Email is invalid"` |
| `"Invalid"` | `"Operating country is required"` |

The field name in the message should match that field's visible label text (what the user
actually reads next to the input), not its internal variable name — e.g. a field whose `FieldLabel`
reads "Full name" gets `"Full name is required"`, even if the underlying schema key is `ownerName`.

## Rule 2 — Within one field, a "required" check always comes first in the validation chain

If a field is both required and has a format/length constraint (email format, max length,
pattern), the "this field is empty" check must be the first check in the chain, so an empty
field always shows a "required" message — never a "this doesn't match the expected format"
message about nothing having been entered at all. Wrong order tells the user "what you typed is
invalid" when they haven't typed anything yet.

```ts
// Wrong — an empty email shows "Email is invalid", not "Email is required"
ownerEmail: z.string().email("Email is invalid").max(320)

// Right — required check runs first in the chain
ownerEmail: z
  .string()
  .min(1, "Email is required")
  .email("Email is invalid")
  .max(320, "Email must be 320 characters or fewer")
```

General ordering within a field's chain: **required → format (email, pattern) → length (min/max)
bounds beyond presence**. A field that's genuinely optional skips the required check entirely
(don't fake one that never fires).

## Rule 3 — Don't rely on the validation library's own default message

Every `.min()`, `.max()`, `.email()`, etc. call takes an explicit message argument. Never leave
a check to fall back to the library's own generated text (e.g. Zod's default `"String must
contain at least 1 character(s)"`) — those are technical, not written for an end user, and don't
name the field.
