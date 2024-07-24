# Koon's Auth

Opinionated Fullstack Expo Authentication

- Expo
- React Native for Web (No Nextjs Support)
- tRPC
- Drizzle
- OAuth
- OTP

<img src="https://i.imgur.com/NoCRvOA.jpeg" height="300px" />

> Pictured here is the most out of shape KoonsAuth developer (very true)

## Login Example

```ts
// packages/auth/index.ts
export const authConfig = config({
    adapter: drizzleAdapter(db),
    providers: {
        google: new GoogleOAuth(...),
        twilio: new Twilio(...),
        resend: new Resend(...),
    }
})
// Idk where this goes
export const auth = makeAuth(authConfig);


// apps/client/app/util.ts
export const { useLogin } = createReactAuth(authConfig);

// apps/client/app/login.tsx
const { login: google } = useLogin("google"); // typesafe, btw
const { login: sms, sendCode: sendText } = useLogin("twilio"); // typesafe
const { login: email, sendCode: sendEmail } = useLogin("resend"); // typesafe

google()
sendCode({ phone: "+18000000000" }) // typesafe
sendEmail({ email: "max@example.com" }) // typesafe

sms("123456", { phone: "+1800000000" }); // typesafe
email("123456", { email: "max@example.com" }); // typesafe

```

## Inside a tRPC protected procedure

```ts
// All the fun stuff happens on ctx.session

// ctx.session.user

// Get the user's id
const id = ctx.session.user.id;

// Query the user's data
const data = await ctx.session.user.get()

// Add an email
await ctx.session.user.resend.addAndSendCode("max@example.com");
// Verify an email
const ok = await ctx.session.user.resend.verify("123567");

// Add a phone number
await ctx.session.user.twilio.addAndSendCode("+18000000000");
// Verify a phone number
const ok = await ctx.session.user.twilio.verify("123567");
```

## Create tRPC procedures based on the authentication providers a user has set up

```ts
const protectedProcedureWithEmail = auth.resend.middleware.isVerifiedEmail(protectedProcedure); // checks for a verified email
const protectedProcedureWithPhone = auth.twilio.middleware.isVerifiedPhone(protectedProcedure); // checks for a verified phone
const protectedProcedureWithSpotify = auth.spotify.middleware.isLinked(protectedProcedure); // checks for an oauth connection to the account

{
    sendEmail: protectedProcedureWithEmail
        .mutation(({ ctx }) => {
            return ctx.session.user.resend.send(ReactComponent({ foo: "bar" }))
        }),
    sendText: protectedProcedureWithPhone
        .mutation(({ ctx }) => {
            return ctx.session.user.twilio.send("Hello there")
        }),
    getPlaying: protectedProcedureWithSpotify
        .query(({ ctx }) => {
            return ctx.session.user.spotify.getCurrentTrack()
        })
        
}

```



## Problems with Existing Auth

- I need phone number OTP, and you need to pay Clerk $30 a month for that.
- I need React Native for Web, Clerk does not support that yet (7/24/2024)


