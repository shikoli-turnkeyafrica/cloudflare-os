# Systems of Agency — The Plain Version

*A no-jargon companion to [THESIS.md](THESIS.md). Same argument, kitchen-table words.*

---

## 1. Software is becoming as cheap as writing a note

Today, when you need to organize something, you open a document and type. You'd never *buy* a product to make one shopping list. Cloudflare OS makes apps work the same way: you tell an AI "make me a dashboard for this," and it does, in seconds, just for you. Your own private copy. If it's missing a button, you ask for the button. You'd never file a "feature request" for your own shopping list — now you don't for your own apps either.

## 2. But there's a catch: a document can't rob you. An app can.

A note on paper just sits there. A program can send emails, move money, leak secrets. This is why companies never let employees write their own software — not because it was hard to write, but because every new program is a new thing that can go wrong. AI just made *writing* software nearly free. It did nothing to make *trusting* it free. That's the actual problem Cloudflare set out to solve, and it's the part of the system with all the clever engineering in it.

## 3. The solution is a chaperone

Between the AI and the real world — your email, your bank, your GitHub — stands a guard (Cloudflare calls them "Gatekeepers"). The guard has two simple rules:

- **Every time the AI reads something, the guard writes it down.**
- **Nothing the AI does actually happens until a human says yes.**

## 4. The genuinely clever trick: the AI doesn't wait for the yes

Normally, "human must approve" means the AI stops and stands there until you click a button. That's so annoying that everyone eventually turns approvals off — which defeats the whole point. So Cloudflare did something sneaky: when the AI "sends" an email, the guard *pretends it was sent*. It shows the AI a fake world where the email exists. The AI, fooled, happily carries on with the rest of its work. Later — tonight, tomorrow — you look at the pile of things the AI *wants* to do and approve or reject them in one sitting.

It's like a kid writing checks that only clear when a parent countersigns — except the kid's own checkbook shows them as already spent, so the kid keeps budgeting and doesn't nag. The AI lives in an imagined world; only your signature makes any of it real. That's the wall between what the AI *imagines* and what actually *happens*.

## 5. What you've seen changes what you're allowed to do

Companies already have this rule for people: attend the secret merger meeting, and suddenly you're not allowed to trade the stock. Not because you changed — because *what's in your head* changed. Cloudflare applies this to AI. Since the guard wrote down everything the AI read, the system knows what's "in its head." If the AI read confidential files and then built a dashboard, you can't share that dashboard with someone who wasn't allowed to see those files.

It even works backwards: if someone's already watching the dashboard, the AI gets *blocked from reading* things that person isn't cleared for — exactly like a classified briefing stopping when the wrong person walks into the room.

## 6. Three questions you'd ask about any worker

Whether the worker is a person or an AI, you want to know:

1. **What are you allowed to do?** ✅ Cloudflare answers this well.
2. **What have you seen?** 🟡 Cloudflare has started answering this.
3. **Did what you did actually work — did the promise come true?** ❌ Nobody asks this. Once an action is approved and fired off, the system never goes back to check that the world matches what was promised. It's a bank that executes every trade and never balances the books at the end of the day.

## 7. The funny part: the checking machinery is already lying around

Remember the fake world from step 4? To fool the AI convincingly, the guard has to know *exactly* what the world should look like after the action. That's a prediction — a precise, machine-readable promise. Right now, the moment the action really runs, the prediction gets thrown away.

All the missing piece requires is: **keep the prediction, then go look, then compare.** Promise versus reality. When they don't match, raise your hand.

## 8. And if you keep score, something big falls out

Count it up over thousands of actions: how often did this AI's promises come true? Now you have a track record — like a driving record. And what do driving records make possible? *Insurance.* An AI with a verified track record is an AI someone can put a price on, post a bond for, insure against.

And that's the moment strangers can trust an AI they've never met — not because they inspected its code, but because someone with money on the line vouches for it. That's how trust in people has always scaled: not inspection — track records and insurance.

---

## The whole thing in one breath

Cloudflare made software cheap like talk, built a chaperone that lets an AI imagine freely while humans control what becomes real, and remembers what the AI has seen so its knowledge limits its actions. The one thing nobody built yet: going back to check the promises came true. Whoever builds *that* turns AI track records into something you can insure — and that's the missing foundation for trusting AI out in the world.
