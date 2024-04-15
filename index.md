
<img src="https://github.com/ohaijen/panza-page/blob/main/panza_logo.png?raw=true" alt="panza logo" width="300"/>

# Panza: A personal email assistant, trained and running on-device.

## What is Panza?
Panza is a tool for automatically generating emails that are customized to your writing style and past e-mail history.
More precisely, we provide process for using your past emails to finetune a publicly-available Large Language Model with 7 billion parameters (we use Mistral-Instruct-7b).
One thing that is special about Panza is that it **can be trained (and run) on a single GPU with
about 22GiB of memory**. This assures privacy, as, generally, there is no need to upload sensitive email data to an external tool or server.
Training is also quick - for a dataset on the order of 1000 emails, training takes well under an hour. The resulting model largely learns the writing style of the original author.

> [INST] Write a message to my local neighborhood mailing list, asking if anyone can water my plants and feed my cat Spike the last week in March.
> [/INST]
>
>   **Panza:**
>
>  Hi everyone,
>
> I'll be away from March 23 to March 30. If anyone can water my plants and
> feed my cat Spike, that would be great.
>
> Thanks,
>
> Jen
>
> **Mistral-7B-Instruct:**
>
>Dear Neighbors,
>
>I hope this message finds you well. I am writing to kindly request your help with taking care of my plants and cat Spike for the last week in March. I will be away on a business >trip and will not be able to come back home to take care of my garden and my furry friend.
>
>If anyone is willing and able to help me out, I would be extremely grateful. Please let me know if you can come by my house and water my plants and feed Spike. I will make sure to leave you with the necessary instructions and supplies.
>
>Thank you in advance for your kindness and generosity. I look forward to hearing from you soon.
>
>Best regards,
>[Your Name]


As an additional bonus, Panza retains many of the capabilities of the original model, allowing for the injection of world knowledge into the generated emails (however, use at your own risk).


>[INST] Write an email to my friend Stew explaining how a steam engine works.
> [/INST] Hi Stew,
>
>Steam engines work by heating water to create steam. The steam is
>then used to push a piston, which in turn moves a crankshaft. The
>crankshaft is connected to the wheel, so when the piston moves, the
>wheel moves, and the train moves.
>
>Best,
>
>Jen


## How it works
For most email clients, it is possible to download a user's past emails in a machine-friendly .mbox format.
These are used as training data, as follows. For each email, Panza uses a pretrained model (we use Mistral-Instruct-7b with no additional training) to write a command that might lead an LLM to generate the email. This gives us a (semi-synthetic) training set that we use to finetune a LLM to generate emails that match those written by the user, in response to the synthetic prompts created in the previous step.

We then use parameter-efficient finetuning to train the LLM on this dataset. We found that we get the best results with the RoSA method, which combines low-rank (LoRA) and sparse finetuning. If parameter efficiency is not a concern, then regular, full-rank/full-parameter finetuning can also be used. We find that a moderate amount of further training strikes the right balance between matching the writer's style without memorizing irrelevant details in past emails.

Optionally, Panza can be run with a Retrieval-Augmented Generation (RAG) module. This functionality stores past emails in a database and provides several of the most relevant ones for each new query. This allows Panza to better insert specific details, such as a writer's contact information or frequently used Zoom links.

# Step-by-step guide

To train your personalized email assistant, follow the three steps below.

1. Follow the README in the `data_preparation` directory to prepare your personalized instruction/email dataset.
2. Change to the `finetuning` directory and follow the tutorial there to finetune a `Mistral-7B` model on your dataset.
3. Finally, in the `serving` directory you can employ your finetuned model to generate emails.
