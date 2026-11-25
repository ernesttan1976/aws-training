# Hints — Lab 01: Reliable asynchronous ticket intake

Use these in order. Stop as soon as one is enough to unblock you.

## Level 1 — Guided implementation

1. **Broad:** Keep the producer and consumer separate. The queue is the boundary that absorbs bursts and lets the consumer retry independently.
2. **Architecture:** An SQS event source mapping polls the queue and invokes Lambda with batches. With batch size 1, each failed invocation is easy to associate with one message.
3. **Failure behavior:** A function error does not acknowledge the message. SQS makes it visible again after the visibility timeout; after `maxReceiveCount`, the redrive policy moves it to the DLQ.
4. **Timeout relationship:** Start with a short Lambda timeout. Set the queue visibility timeout to at least six times that timeout, then verify the configured values in AWS after deployment.
5. **Permissions:** The producer needs `sqs:SendMessage` on the work queue. The Lambda execution role, which the event source mapping uses to read the queue, needs the required receive/delete/attribute actions and log permissions. Avoid `sqs:*` and `Resource: "*"` when a queue ARN can be specified.
6. **SDK setup:** Let the AWS SDK use the standard credential provider chain. Select the intended CLI profile in your shell; do not embed credentials in code or `.env` files committed to Git.
7. **Validation:** Make invalid input fail in the consumer. Use a deliberately small receive threshold so you can observe DLQ movement quickly, while allowing enough time for each visibility timeout.
8. **Deployment review:** Inspect the SAM template and change set before creating resources. A VPC attachment or NAT gateway is outside this lab's scope and can change the cost materially.

## Questions to answer while working

- Which component acknowledges successful work, and when?
- What retries a failed invocation: your producer, Lambda, or the queue/event-source integration?
- What is the difference between a message being invisible and being deleted?
- What evidence would distinguish “Lambda never invoked” from “invoked and failed”?

Level 2–4 hints will be added when those exercises become active; they are not part of the current assignment.
