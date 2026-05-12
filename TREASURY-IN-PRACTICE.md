# Treasury in Practice: A Practitioner Vignette

*This document is a practitioner vignette — an illustrative aside, in the tradition of
AFP Exchange editorial commentary and ETM8 boxed examples, that grounds the analytical
framework of this series in the lived experience of a treasury professional. The character
is fictional. The methodology she applies is the methodology documented in C001 through
C014. The two are presented together because the framework, stripped of the person who
holds it, is harder to remember than it should be.*

*Gladys is nobody's intellectual property — least of all ours. We make no claim to the
character, the archetype, or the particular brand of professional commitment she
represents. Any resemblance to a working CTP practitioner is plausible by design.*

---

## Gladys, CTP: An Illustrative Profile

---

## Case Study Overview

**Subject:** Gladys, CTP
**Function:** Corporate Treasury Manager
**Specialty:** Cash and Liquidity Management, Float Reduction, Payment Rail Optimization
**Tenure:** Fifteen years. She has not taken a full week of PTO in six of them.
**Clinical observation:** Gladys presents a textbook application of AFP/CTP methodology
across all treasury domains. She also presents a textbook illustration of what happens
when a person's professional framework becomes indistinguishable from their personality.
Both presentations are instructive. Only one of them is in ETM8.

This case study is offered in two parts. Part I is the professional analysis — how Gladys
applies the AFP/CTP framework, why her approach is correct, and what treasury functions
operating without it are missing. Part II is Gladys's own blog, which she started because
her therapist suggested journaling, and which has become, primarily, a float journal.

Her therapist has noted this.

---

## Part I: The Professional Case

Gladys operates a treasury function with a level of analytical precision that most
practitioners would describe as exceptional and a smaller number would describe as
alarming. The distinction between these two characterizations tends to correlate with
how recently the observer has had to explain a float variance to her.

**On float management:** Gladys applies the AFP float taxonomy without exception.
Friction float — collection float, availability float, processing float — is the enemy
and is treated as such. Strategic float — clearing float, disbursement float — is
scheduled, not suffered. Her net float position is computed daily. She considers weekly
computation a governance gap. She is, by the standards of ETM8 Chapter 12, correct.

**On payment rail selection:** Gladys does not have payment preferences. She has payment
decisions, made analytically, documented with disbursement cost analysis, and reviewed
quarterly for drift against the ECR. When someone suggests Fedwire for a payment that
could be same-day ACH, she does not express frustration. She opens a spreadsheet. The
spreadsheet, over time, has become its own form of expression.

**On the investment policy statement:** Gladys's IPS is current, specific, and enforced.
The WAM ceiling is monitored in real time. Counterparty concentration is reviewed at each
deployment. Credit tier compliance has never lapsed on her watch. She considers a lapsed
IPS constraint to be a personal failure roughly equivalent to a public health crisis, in
that both are preventable and both reflect badly on whoever was supposed to be paying
attention.

**On the digital asset question:** Gladys was asked, at a conference, whether she was
excited about cryptocurrency. She said: "The question is whether the treasury function
governing digital assets meets the same professional standards as the treasury function
governing traditional instruments. In most cases I have reviewed, it does not. That is
a solvable problem. The AFP/CTP framework provides the solution." The questioner had been
hoping for something more. Gladys had provided everything relevant.

She is right about all of this. That is established. Part II follows.

---

## Part II: Gladys's Blog

*"My therapist said I should write things down. I write things down constantly —
that is not the issue — but she means feelings, not float positions. I have decided
these are the same thing. I am starting a blog."*
— Gladys, first entry

---

### Entry 1: On the Nature of Float

I am going to explain float because I find that most people have not had it explained
to them correctly and this explains a great deal about the state of the world.

Float is the time between when money leaves your control and when it arrives somewhere
useful. During that time, the money exists in a kind of financial purgatory — present
on the ledger, absent from the investable balance, earning nothing for anyone, costing
you ECR opportunity at the current annual rate divided by 365 for every day it sits there.

This bothers me. It has always bothered me. When I was eight years old and my mother
gave me lunch money, I would think about the fact that the money was not earning anything
in my pocket and I did not yet have a framework for why this bothered me, only that it
did. I now have the framework. It is called the AFP Body of Knowledge and I have read it
completely. The money in my pocket still bothers me but now at least the feeling has a name.

Float is reducible. This is the important thing. It is not a force of nature. It is a
measurement of process efficiency, and process efficiency is improvable. I find this
deeply comforting. My therapist says she is glad I find something comforting and asked
if I could also find comfort in things that do not involve basis points. I told her I
would think about it.

I have not thought about it. There was a same-day ACH cutoff at 11:30 and then the
afternoon batch and then the month-end close and then I found a $340 earnings allowance
variance and it was availability float from a misdirected batch submission and I fixed it
and the float position is correct now and I feel fine. Better than fine. I feel the way
I imagine people feel after exercise, which I have been told is also good for me.

The float is reduced. I am well. More tomorrow.

---

### Entry 6: I Am Not Well

I want to be honest about something.

I am not feeling well. It has been a difficult week. The investable balance has been
performing below projection for four consecutive days due to an availability schedule
discrepancy I have not been able to fully isolate — I have narrowed it to either a
same-day window submission error or a change in the bank's availability schedule that
was not communicated — and the uncertainty is, frankly, distressing. I have run the
BAA worksheet three times. The numbers change each time I update the outstanding float
estimate. This should not happen. I do not like things that should not happen.

My assistant asked if I was okay. I said the investable balance is off by $8,200 and
I am not okay until I know why. She brought me a cup of coffee. I thanked her and then
I mentioned that if we submitted the next batch at the W1 opening instead of waiting
for W2 we could recover approximately $47 in float earnings for the day. She left.

I think she was trying to be kind. The coffee was good. The float remains unreduced.

The bank's daily account analysis statement arrived at 4:47pm. I read it immediately.
The variance is availability float from a Tuesday return item that did not clear until
Thursday — an availability delay I had not modeled because return item clearing times
are not documented in our current SLA. I updated the SLA documentation. I flagged the
item as a process exception. I computed the ECR opportunity cost: $23.

I am going to make sure this does not happen again.

I feel much better.

---

### Entry 12: The Digital Asset Question

Someone at a conference today told me that blockchain eliminates float.

I considered this statement for longer than I normally consider things, because I wanted
to be fair to it before I explained why it is not accurate.

It does not eliminate float. It changes the unit in which float is measured. On-chain
transactions have confirmation depth. Confirmation depth has duration. Duration in which
funds are committed but not settled is float. It is protocol-defined float with a
different name, and in some networks it is auction-priced float, which is the version of
float I find most distressing because you cannot run a reliable BAA on a variable you
cannot forecast.

I said this. The person who had told me blockchain eliminates float did not respond in the
way I expected, which is to say they did not begin updating their investable balance model.
They said "huh." I believe this represents a professional development opportunity.

The part that keeps me up at night — and I want to be honest, several things keep me up
at night, but this is on the list — is that there are treasury functions operating on
digital asset rails right now where the practitioner does not know their clearing epoch,
does not have an availability schedule, has not computed a float position, and is making
disbursement decisions without a payment cost analysis. The money is moving. The framework
is not there. The float is uncontrolled.

This is not acceptable to me. It is not acceptable in the same way that I find it not
acceptable when someone uses Fedwire for a payment that could be same-day ACH. The
principle is the same. The costs are larger.

I came home and updated my IPS.

I feel somewhat better.

---

### Entry 17: On Assistance

I would like to address, briefly, the characterization of my management style as
"demanding."

Treasury management is a precise discipline. Precision requires information at the right
time in the right format. When I need the outstanding float detail before the 8am meeting,
I need it before the 8am meeting — not during, not after, and not "approximately the
outstanding float detail" with a note that the exact figures will follow. The exact figures
are the point. I have explained this.

My assistant is talented. She pulls the detail quickly. She has learned, over two years,
how I think about the BAA worksheet, what the thorn register is for, and why the same-day
ACH window schedule is not optional context but structural information. This is professional
development. She is becoming a better treasury analyst. I consider this a management
success.

She has, I notice, started calling the settlement exception log the "thorn register"
without prompting. I consider this a milestone.

She also gave me soup last Thursday when I mentioned that I had not been sleeping well.
I do not know how she knew that soup was what was needed. I told her the availability
float analysis was keeping me up. She said she thought it might be. She brought the soup
anyway.

The float was resolved by Friday. The soup was vegetable. Both were good.

---

### Entry 23: A Reflection on What This Is All For

Someone asked me — my therapist, actually — why I care so much about float.

The honest answer, which I have been thinking about for three weeks and which I am now
prepared to state: because the money belongs to someone.

Float is not an abstraction. It is the gap between when an obligation is met and when
the beneficiary has the value. That gap costs something. In commercial treasury, it costs
ECR. In personal finance, it costs opportunity. In institutional operations at scale,
it costs real money — not basis points of it, real dollars, owed to real counterparties
who made decisions based on when they expected the funds to arrive.

When float is uncontrolled, the counterparty pays the cost of someone else's process
failure. When float is managed well, the cost goes to zero and the money is where it
is supposed to be when it is supposed to be there. This sounds simple. It is not simple
to execute. It is very simple to understand why it matters.

I have been doing this for fifteen years. I am doing it in the digital asset space now
because the money is still the money, the obligation is still the obligation, and the
float is still reducible. Nobody has suspended the AFP/CTP framework because the
instrument changed. The instrument changed. The framework is the same.

My therapist said that was a good answer and asked if I could apply similar reasoning
to other areas of my life.

I have a lot of thoughts about float in other areas of my life. I did not think that
was what she meant.

She asked a follow-up question, which I was not expecting. She asked: when you find
a float variance — when you trace it, isolate it, resolve it — what does that feel like?

I said: correct. It feels correct.

She said: is that the same as good?

I thought about this for a while. I told her that I have a thorn register. It is a log
of settlement exceptions — items where the on-chain state and the float register
disagree, or where the clearing epoch elapsed and the position did not settle. Every
item on that register is an obligation that has not reached its counterparty on schedule.
Every time I resolve one, I close it. The register gets shorter. The money is where it
is supposed to be.

She asked if I could describe the feeling when I close an item on the thorn register.

I said it feels like something was wrong and now it is not wrong anymore. I said it feels
like the gap closed. I said it feels like — and here I had to stop, because I do not
generally describe things as feeling like anything, and I was aware that I was about to —
it feels like keeping a promise that I made to someone I have never met and will never
meet, on their behalf, using a spreadsheet.

My therapist was quiet for a moment.

She said: I think that is what care looks like for you.

I told her I was going to need to think about that. She said she thought I probably would.

I have been thinking about it. The thorn register. The BAA worksheet. The same-day ACH
cutoff I know by heart because missing it costs someone something. The payment rail
analysis I run quarterly not because policy requires it but because I would know if the
ECR shifted and I had not updated the model. The IPS constraints I monitor in real time
because a lapsed constraint is an obligation I made to the portfolio and did not keep.

I think she is right. I think these are all the same thing. I think I have been doing
this — all of this — because I cannot tolerate the gap. Not the float gap. Any gap.
The distance between what should be and what is. The space where something owed has
not yet arrived.

I reduce float because the beneficiary on the other end of the transaction is real,
made decisions based on when the funds were supposed to arrive, and deserves the
precision I am capable of giving them. The instrument is irrelevant. The obligation
is not. I know how to close the gap. So I do.

My therapist said that was the most useful thing I have said in two years of sessions.

I said the thorn register currently has zero open items.

She said she knew.

More tomorrow. There is a month-end close.

---

*Gladys is a thought experiment. The treasury management principles she applies with
a ferocity that concerns her colleagues and, increasingly, herself are real, extensively
documented, and codified by the Association for Financial Professionals. The float she
reduces is fictional. Yours is not.*

*If you are a treasury professional reading this and you recognized yourself: the float
is reducible. The BAA worksheet is in C008. You are going to be fine.*

*If you are not a treasury professional and this is your introduction to float: welcome.
It is a lot. We are glad you are here. Please read C001 first.*
