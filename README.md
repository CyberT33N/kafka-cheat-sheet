# kafka-cheat-sheet
Kafka with the most needed stuff..









# avro

<br><br>
<br><br>

## Schema
- https://www.npmjs.com/package/avsc
- https://avro.apache.org/docs/1.11.1/specification/











<br><br>
<br><br>
_______________________________________
_______________________________________
<br><br>
<br><br>


# kafka.js





<br><br>
<br><br>

## DOCS
- [https://kafka.js.org/docs](https://kafka.js.org/docs/getting-started)










<br><br>
<br><br>

## Admin
- https://kafka.js.org/docs/1.11.0/admin

<br><br>

### fetchTopicMetadata()
- Fetch metadata of specific topic
```javascript
const res = await kafkaClient.admin.fetchTopicMetadata({
    topics: [testTopics.thumbnail]
})
```

<br><br>

### deleteTopicRecords()
- Delete records in topic
```javascript
await kafkaClient.admin.deleteTopicRecords({
    topic: 'topicName',
    partitions: [
        { partition: 0, offset: '-1' }, // delete all available records on this partition
    ]
})
```






















<br><br>
<br><br>
_______________________________________________________________
_______________________________________________________________
<br><br>
<br><br>

# send
```javascript
'use strict'

const { Kafka } = require('kafkajs')
const { AisAvroMessageConverter } = require('ais-kernel-kafka')
const { v4: uuid } = require('uuid')

/**
 * Represents a KafkaHelper.
 */
class KafkaHelper {
    /**
     * Constructs a new KafkaHelper instance.
     */
    constructor() {
        this.broker = 'localhost:9092'
        this.clientId = `KafkaHelper-${uuid()}`
    }

    /**
     * Creates a Kafka producer.
     */
    async _createProducer() {
        try { 
            const kafka = new Kafka({
                clientId: this.clientId,
                brokers: [this.broker]
            })
        
            this.producer = kafka.producer()
        } catch (e) {
            throw new BaseError('_createProducer() - Error while create producer', e)
        }
    }

    /**
     * Connect to producer
     */
    async connect() {
        if (!this.producer) {
            await this._createProducer()
        }

        try { 
            await this.producer.connect()
        } catch (e) {
            throw new BaseError('connect() - Error while connect to producer', e)
        }
    }

    /**
     * Sends a message to a topic.
     * @param {string} msg - The text of the message.
     * @param {string} topic - The name of topic.
     */
    async sendMessage(msg, topic) {
        if (!this.producer) {
            await this.connect()
        }

        try {
            await this.producer.send({
                topic,
                messages: [
                    {
                        value: msg
                    }
                ]
            })
        } catch (e) {
            throw new BaseError('sendMessage() - Error while sending message to topic', e)
        }
    }

    /**
     * Sends an Avro message to a topic.
     * @param {object} body - The body of the message.
     * @param {string} topic - The name of the topic.
     * @param {object} headers - The headers of the message.
     * @param {object} options - The options for sending the message (optional).
     */
    async sendMessageAvro(body, topic, headers = {}, options = {}) {
        if (!this.producer) {
            await this.connect()
        }

        const messages = [{ value: AisAvroMessageConverter.convertToAvro(headers, body) }]

        try {
            await this.producer.send({ topic, messages })
        } catch (e) {
            throw new BaseError('sendMessage() - Error while sending message to topic', e)
        }
    }
}

module.exports = KafkaHelper
```

```javascript
/**
 * 
 * @param {*} testRequirements 
 */
module.exports = testRequirements => {
    with (testRequirements) {
        describe('[UNIT] - KafkaHelper', () => {
            let kafkaHelper

            beforeEach(() => {
                kafkaHelper = new KafkaHelper()
            })
      
            afterEach(() => {
                sinon.restore()
            })
      
            describe('constructor', () => {
                it('should initialize broker and clientId', () => {
                    expect(kafkaHelper.broker).to.exist
                    expect(kafkaHelper.clientId).to.equal('KafkaHelper')
                })
            })

            describe('_createProducer()', () => {
                it('should create producer', async () => {
                    await kafkaHelper._createProducer()
                    expect(kafkaHelper.producer).to.exists
                })
            })
      
            describe('connect()', () => {
                it('should connect to Kafka producer', async () => {
                    const createProducerSpy = sinon.spy(kafkaHelper, '_createProducer')

                    const producerMock = {
                        connect: sinon.stub().resolves()
                    }
 
                    kafkaHelper.producer = producerMock

                    // _createProducer() should be not called because we overwrite this.producer
                    await kafkaHelper.connect()

                    expect(createProducerSpy.calledOnce).to.be.false
                    expect(kafkaHelper.producer).to.exists
                    expect(producerMock.connect.calledOnce).to.be.true
                })
            })
      
            describe('sendMessage()', () => {
                let producerMock

                const msg = 'Test message'
                const topic = process.env.AIS_KAFKA_DLQ_TOPIC
      
                beforeEach(() => {
                    producerMock = {
                        connect: sinon.stub().resolves(),
                        send: sinon.stub().resolves()
                    }

                    kafkaHelper.producer = producerMock
                })
      
                it('should send message to topic', async () => {
                    const connectSpy = sinon.spy(kafkaHelper, 'connect')

                    // connect() should be not called because we overwrite this.producer
                    await kafkaHelper.sendMessage(msg, topic)
                    expect(connectSpy.calledOnce).to.be.false
                    expect(producerMock.send.calledOnce).to.be.true
                    expect(producerMock.send.firstCall.args[0]).to.deep.equal({
                        topic,
                        messages: [{ value: msg }]
                    })
                })
      
                it('should throw an error if sending message fails', async () => {
                    const error = new Error('Message sending failed')
                    sinon.stub(kafkaHelper, 'connect').resolves()

                    producerMock.send.rejects(error)

                    await expect(kafkaHelper.sendMessage(msg, topic)).to.be.rejectedWith('sendMessage() - Error while sending message to topic')
                })
            })

            describe('sendMessageAvro()', () => {
                let producerMock

                const body = {
                    "originalBody": 'test',
                    "errorMessage": 'Some error..',
                    "errors": ['Any error..']
                }

                const headers = {
                    'project-id': projectId,
                    'mailing-id': 'test'
                }

                const topic = 'test'
      
                beforeEach(() => {
                    producerMock = {
                        connect: sinon.stub().resolves(),
                        send: sinon.stub().resolves()
                    }

                    kafkaHelper.producer = producerMock
                })
      
                it('should send avro message to topic', async () => {
                    const connectSpy = sinon.spy(kafkaHelper, 'connect')

                    // connect() should be not called because we overwrite this.producer
                    await kafkaHelper.sendMessageAvro(body, topic, headers)
                    expect(connectSpy.calledOnce).to.be.false
                    expect(producerMock.send.calledOnce).to.be.true
                    expect(producerMock.send.firstCall.args[0].messages[0].value instanceof Buffer).to.be.true
                    expect(producerMock.send.firstCall.args[0].topic).to.be.equal(topic)
                })
            })
        })
    }
}
```










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
