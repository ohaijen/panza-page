## Why Panza?

A digital personal assistant is software that helps manage various daily tasks such as email, calendar, writing, summarizing, recalling past events or actions, etc, that take up unneeded time in today’s laptop and phone environments. One approach to build such an assistant would be based on a Large Language Model (LLM). An ideal assistant would have the ability to:

- **Personalize** the model, that is, customize it to the specific individual. 
- **Protect the user’s privacy**, that is, allow the AI to have access to very personal information that many people will never agree to share, even if promised that this will be protected by the provider’s cloud.
- **Have low overhead**, that is, not require extensive work by the user to build or improve. Ideally, one would like to avoid tedious tasks such as manual data creation or annotation, and extensive prompt engineering.

Today's LLM-based assistants typically reside in the cloud, and are accessed via an API. This has the advantage of simplicity and the ability to use the capabilities of the latest large models. However, this approach makes it computationally expensive and difficult to reliably and credibly meet the first two goals stated above.

Panza is a desktop LLM-based assistant that aims to tackle these challenges jointly. In Panza, all data and custom model weights stay local, allowing the user to confidently provide Panza with sensitive information. This access will allow Panza to mimic the user's expression, avoiding the 'corporate' or 'sterilized' feel of commercially available models. At the same time, all data curation and annotation steps are automated, minimizing the overhead of training and deploying Panza. We envision Panza as continuously based on the data of a specific user, learning the history, the associativity patterns, and the style of the user, and using this data to deliver a great individual experience.

Our first version of Panza is Panza Mail.

## What is Panza Mail?
Panza Mail is a tool for automatically generating emails that are customized to your writing style and past e-mail history.
More precisely, we provide a process for using your past emails to finetune a publicly-available Large Language Model (we use Mistral-7B-Instruct as a base model).
One thing that is special about Panza is that it **can be trained (and run) on a single GPU with
about 22GiB of memory**. This assures privacy, as, generally, there is no need to upload sensitive email data to an external tool or server.
Training is also quick - for a dataset of about 1000 emails, training takes about an hour. The resulting model largely learns the writing style of the original author.
We plan to also release more efficient versions of Panza that would require less GPU memory or run entirely on CPU within reasonable time. 

Here is an example of Panza in action, compared with the base model: 

> [User Instruction]  Write an email to my dad with my new address and phone number.

Panza:

> Hi Dad,
>
> I hope you're doing well! I wanted to share my updated contact information with you:
> 
> Address:  Schönbrunner Schloßstraße 47, 1130, Vienna, Austria
> 
> Phone Number: +43 01 81113239
> 
> I'm looking forward to catching up with you soon. Let me know if there's a good time for you to call.
> 
> Best,
> 
> Jen

Mistral-7B-Instruct:

> Dear Dad,
>
> I hope this email finds you well! I'm excited to share some news with you - I have recently moved to a new place and I'd be happy to provide you with my updated contact information.
>
> My new address is:
> 
> [Your Name]
> 
> [New Address]
> 
> [City, State, Zip Code]
>
> 
> And my new phone number is:
> 
> [Your Name]
> 
> [New Phone Number]
> 
>
> I'd love to hear from you soon and catch up on all the latest happenings. If you have a moment, could you please update your records with my new contact information? I'd appreciate it!
>
> I'm looking forward to settling into my new place and exploring my new neighborhood. I'll be sure to send you pictures and updates as I make progress.
> 
> Take care and give my love to Mom and the rest of the family.
>
> Best,
> 
> [Your Name]</s>


As an additional bonus, Panza retains many of the capabilities of the original model, allowing for the injection of world knowledge into the generated emails (note that in the email below, Panza correctly inferred that machine learning was involved).


>[User Instruction] Write an enthusiastic e-mail to Armand presenting the new writing assistant called Panza.
>
>
> Hi Armand,
>
>Super excited to share with you the new writing assistant I've been working on! It's called Panza and it's going to make our lives so much easier.
>
>It's a machine learning model that can understand our writing style and can help us write emails more efficiently.
>
>I've been testing it for a few days now, and it's already making a huge difference. It's like having a personal assistant that can write emails for us.
>
>I think it's going to be especially helpful for us, given the number of emails we have to write every day.
>
>Let me know what you think!
>
>Best,
>
>Dan


## How it works 

<img src="https://github.com/ohaijen/panza-page/blob/main/panza_diagram.png?raw=true" alt="panza diagram"/>

### Step 1: Data playback

For most email clients, it is possible to download a user's past emails in a machine-friendly .mbox format. For example, GMail allows you to do this via [Google Takeout](takeout.google.com). 
We translate sent emails locally into training data for Panza, as follows. For each email, Panza uses a pretrained model (we use [Mistral-Instruct-7b](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2) with no additional training) to write an instruction that should lead a customized personal assistant LLM to generate the email. This gives us a (semi-synthetic) training set of (instruction; true email) pairs that we use to finetune a LLM to generate emails that match those written by the user, in response to the synthetic prompts created in the previous step. We call this technique data playback. 

### Step 2: Local Fine-tuning via RObust Adaptation (RoSA)

We then use parameter-efficient finetuning to train the LLM on this dataset, locally. We found that we get the best results with the [RoSA method](https://arxiv.org/pdf/2401.04679.pdf), which combines low-rank (LoRA) and sparse finetuning. If parameter efficiency is not a concern, that is, you have a more powerful GPU, then regular, full-rank/full-parameter finetuning can also be used. We find that a moderate amount of further training strikes the right balance between matching the writer's style without memorizing irrelevant details in past emails.

### Step 3: Serving via RAG

Once we have a custom user model, Panza can be run locally together with a Retrieval-Augmented Generation (RAG) module. Specifically, this functionality stores past emails in a database and provides a few relevant emails as context for each new query. This allows Panza to better insert specific details, such as a writer's contact information or frequently used Zoom links.


## Try it out

To train your personalized email assistant, please go to the [code](https://github.com/IST-DASLab/panza-dev/tree/master) and follow the instructions in the README. You will need:
1. Your emails, exported to `mbox` format.
2. A computer, preferably with an NVIDIA GPU with at least 22 GiB of memory. We are planning to release a CPU-only version of Panza, but training only on CPU will take considerably longer.
3. Basic Python/unix knowledge, such as building environments and running python scripts.
4. If you plan to use the Mistral models (or most other viable alternatives), a HuggingFace account.

## Authors

Panza was conceived by Nir Shavit and Dan Alistarh and built by the [Distributed Algorithms and Systems group](https://ist.ac.at/en/research/alistarh-group/) at IST Austria. The contributors are (in alphabetical order):

Dan Alistarh, Eugenia Iofinova, Eldar Kurtic, Armand Nicolicioiu, Mahdi Nikdan, Andrei Panferov, and Nir Shavit.

Contact: dan.alistarh@ist.ac.at

We thank our collaborators Michael Goin and Tony Wang at NeuralMagic and MIT for their helpful testing and feedback.
