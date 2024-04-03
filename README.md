# kafkajs-cheat-sheet
Kafka.js with the most needed stuff..





<br><br>
<br><br>

# DOCS
- [https://kafka.js.org/docs](https://kafka.js.org/docs/getting-started)







<br><br>
<br><br>

## Admin
- https://kafka.js.org/docs/1.11.0/admin






<br><br>
<br><br>

## Send message to topic
```javascript
const { Kafka } = require('kafkajs');

async function produceMessage() {
  try {
    const kafka = new Kafka({
      clientId: 'my-app',
      brokers: ['localhost:9092'] // Füge hier die Adresse deines Kafka-Brokers ein
    });

    const producer = kafka.producer();

    await producer.connect();

    await producer.send({
      topic: 'dein-topic-name',
      messages: [
        {
          value: 'Hier ist deine Nachricht.' // Hier kannst du deine Nachricht einfügen
        }
      ]
    });

    console.log('Nachricht erfolgreich gesendet.');

    await producer.disconnect();
  } catch (error) {
    console.error('Fehler beim Senden der Nachricht:', error);
  }
}

produceMessage();
```
