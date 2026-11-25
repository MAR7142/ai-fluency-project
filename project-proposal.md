Degree level: Bachelor’s
# Project proposal

## 1. The problem and goal

(Explain the problem you want to solve and why it matters. Describe what you
want to improve, how you will measure improvement, and which limits or
requirements must be respected. A short example can help someone outside your
field understand the problem.)

**The problem and goal in my project:**

Liver T1 mapping is an MRI measurement used as a marker of liver disease. Our
lab measures it with the Liver Open-MOLLI sequence. The measured T1 is biased
by fat, by tissue T2, by the strength of the radio-frequency field (B1), and by
details of the sequence itself. A dictionary of simulated signals can correct
this bias, but only if the model behind it includes every effect that matters.
Finding the missing effects is slow expert work: when I validated my simulator,
it took me many rounds to discover that the real inversion pulse and the slice
profile were essential.

Goal: build an agent that finds what the model is missing and fixes it. It
does this by testing ideas in the simulator, explaining the errors, and
choosing which physical effect to add next.

How I will measure success:
1. Main test, hidden effects (synthetic data): I generate "scans" from the
   simulator with an effect that the agent's model does not include, such as
   uneven B1 or fat. Several effects could plausibly explain the error, so the
   data must decide. Does the agent identify the missing effect?
2. Second test, rediscovery (real data): starting from a simple model and
   existing phantom scans with known reference values, does the agent find
   the effects I found by hand, and in how many rounds?

Final measure: the T1 error against the true or reference values on a
held-out test set, and, in the main test, how often the agent names the
correct missing effect. Both are compared with a baseline: a simple search
that only tunes numbers in the simple model.

Limits: the sequence is fixed, everything runs on a laptop, and no free
parameters may be added just to match the reference. The agent tunes on one
data set and is tested on another.

## 2. The simulator

(Describe a program or model that could simulate your system and calculate what
happens under different conditions. Explain what inputs it takes and what
results it produces. State whether it can run on your computer and accept inputs
in a text file, such as JSON or YAML. If you have not chosen a simulator yet,
name an option you want to explore and explain what you need to check.)

**The simulator in my project:**

I use two simulators:
- MR-Zero (MRzeroCore): an open-source Python MRI simulator, used as the
  reference.
- My own fast simulator: it follows a single voxel through the exact sequence
  from the Pulseq file. In my tests it matched MR-Zero closely and ran about
  1000 times faster, which makes many rounds of testing possible.

Each physical effect can be switched on or off in a YAML settings file: the
real inversion pulse, the slice profile, T2, B1 and fat.

- Input: the sequence file and the tissue and model settings.
- Output: simulated signal curves and the T1 that the scanner's fit would
  report.

Both simulators run on my laptop.

## 3. The agent's role

(Explain what the language-model-based agent will decide or change, and how it
will use the simulator's results to choose its next action. If known, list the
variables it may change and their allowed ranges. Explain what you expect the
language model to contribute: could a simple program that tries different
options and picks the best result achieve the same goal? Include one question
you still need to answer.)

**The agent in my project:**

In each round the agent:
1. Runs the simulator.
2. Compares the result with the reference data.
3. Runs small diagnostic tests, such as sensitivity to one parameter or a
   convergence check.
4. Explains what the error pattern suggests and proposes one change to the
   model.

I approve each change before it is applied.

What it may change: which physical effects are included, how they are modeled,
and the dictionary ranges and step sizes.

Why a language model is needed: a simple search can only tune numbers. It can
make the simple model match one data set, but only by using physically wrong
values, and those values fail on a second set. Choosing which effect is missing
requires linking error patterns to physics. For example, an error that grows
as T2 gets shorter points to the inversion pulse, and an error that changes
across the image points to B1. The agent can use its physics knowledge to
suggest candidate effects. That is its advantage. The main test then checks
that it uses the data, not only prior knowledge, to choose between them.

Stretch goal: if time allows, the agent will investigate errors that remain
unexplained in my current model, such as a small bias that remains for long-T1
phantoms. A new effect it proposes is accepted only if I confirm that it is
physically sound, it improves results on a held-out set, and it adds no free
parameters.

Open question: how much physics knowledge should the agent's instructions
contain, so it can find the effects without being told the answer?

## Explain your project to someone outside your field

(Now describe your whole project again, as if you were explaining it to someone
from another field who has no background in your topic. For example, if you
study chemical engineering, imagine explaining it to a physicist; if you study
aeronautical engineering, imagine explaining it to someone in medicine; if you
study data science, imagine explaining it to a materials engineer. In one or two
short paragraphs, connect all three parts: the problem you want to solve, how
the simulator helps you test ideas, and what the agent will decide or change.
Use everyday language, explain any technical terms you need, and avoid
unexplained abbreviations. Do not simply copy your answers above.)

**My project in simple words:**

An MRI scanner can measure how quickly liver tissue recovers after a magnetic
pulse, and this helps doctors detect liver disease. But the reading is
distorted by fat and by hidden details of the scan, so it can hide disease or
make a healthy liver look sick. A computer model of the scanner can undo this
distortion, but only if the model knows about every hidden detail, and finding
them by hand took me many attempts.

My project is an AI assistant that does this detective work. It runs the
computer model, compares the results with cases where the true values are
known, figures out why the model is wrong, and suggests what to add. I test it
in two ways. First, on made-up data where I planted a hidden problem myself,
I check whether it finds that problem. Second, on real measurements, I check
whether it rediscovers what I found by hand. I compare it with a simple
program that only adjusts numbers and does not understand why the model is
wrong. If time allows, I will also let it look for causes of errors that I
have not explained yet, and I will check its ideas myself.
