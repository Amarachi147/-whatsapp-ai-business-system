A WhatsApp AI assistant for a fictional car dealership (Prime Motors Naija), built as a portfolio demo and running on a real WhatsApp number. It is a complete system, not just a chatbot: it answers from a knowledge base, books appointments, logs every conversation, hands over to a human, and alerts the owner when something breaks.

This is a demo project with made-up business data. It is not a real client project.

Demo video: [add link] Built by: Amarachi Owoh | [LinkedIn](add link)

The problem

When a business automates its WhatsApp, the owner usually loses visibility. The bot replies, but the owner cannot see what customers ask, when the bot is stuck, or when something has failed.

What the system does
Answers from a knowledge base (RAG): car details, prices, location, payment, delivery and policies come from a vector database, not from guesses.
Books inspections: checks Google Calendar for a free slot, then creates the event.
Logs everything in Airtable: customers, full conversations and escalations, so the owner can see what happened without opening WhatsApp.
Human handoff: when a customer asks for the owner, wants to negotiate, is upset, or the bot cannot answer, the bot steps aside, marks the customer as "Human", and alerts the owner with the customer's name, number, reason and a one-tap WhatsApp chat link.
Stays silent during handoff: while a customer is marked "Human", the bot does not reply. Their messages are forwarded to the owner.
Error handling: if the AI step fails, the customer gets a polite fallback message and the case is escalated. A separate error workflow alerts the owner on WhatsApp if any workflow fails.
Architecture
WhatsApp message
      |
 WhatsApp Trigger -> Extract Message -> Text only?
      |
 Upsert customer in Airtable (read Status)
      |
 Status = Human? --yes--> Log message -> Forward to owner
      |
      no
      |
 AI Agent (OpenAI + memory)
   tools: Knowledge Base (Pinecone) | Check Availability | Create Event
   output: { reply, escalate, reason, customer_name }
      |
 Send reply -> Log conversation -> Needs human?
                                       |
                    yes: set customer to Human -> log escalation -> alert owner
      
 AI error -> fallback message -> same escalation path

Separate workflow: Error Trigger -> alert owner on WhatsApp
Separate workflow: Google Drive file -> Pinecone (loads the knowledge base)
Tech stack
n8n (workflow automation)
WhatsApp Cloud API
OpenAI (chat model and embeddings)
Pinecone (vector database)
Airtable (customers, conversations, escalations)
Google Calendar and Google Drive
Repository contents
README.md
workflows/
  whatsapp_business_system.json    Main bot workflow
  system_error_alert.json          Error alert workflow
  knowledge_base_loader.json       Loads the knowledge base into Pinecone
knowledge_base/
  prime_motors_knowledge_base.md   Sample knowledge base (fictional data)
screenshots/
  canvas.png, airtable.png, owner-alert.png, error-alert.png
Setup
Airtable: create a base with three tables.
Customers: Phone, Name, Status (single select: Bot, Human)
Conversations: Phone, CustomerMessage, BotReply, HandledBy (Bot, Human), Created
Escalations: Phone, Reason, Handled (Yes, No), Created
Pinecone: create an index whose dimension matches your embeddings model, and use the same model for loading and retrieval.
n8n: import the workflow files and reconnect your credentials (WhatsApp, OpenAI, Airtable, Pinecone, Google).
Replace the placeholders: YOUR_AIRTABLE_BASE_ID, YOUR_PHONE_NUMBER_ID, OWNER_WHATSAPP_NUMBER.
Run the knowledge base loader once, then publish the main workflow and set the error workflow in its settings.

Note: WhatsApp only allows free-text messages to a number that messaged the bot in the last 24 hours. For production owner alerts, use an approved message template.

Lessons learned
A green tick does not mean it worked. My knowledge base returned nothing for hours. The cause was one setting: the data loader was set to JSON instead of Binary, so it stored file metadata instead of the document text. Next improvement: validate loader output with a test search after every load.
A hung step never fails, so an error alert never fires. Adding timeouts turns a hang into a failure the alert can see.
Matching settings matter. The embeddings model and dimensions must be identical when loading and retrieving.
Handoff needs state. Marking the customer as "Human" in the database is what keeps the bot from replying over the owner.
Possible next steps
Daily summary sent to the owner
Message templates for owner alerts outside the 24-hour window
Ingest validation with a test query
Heartbeat checks for scheduled jobs
License

For portfolio and learning purposes.
