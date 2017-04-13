# Eclipse Environment

1) `mvn eclipse:eclipse`

## Meaningful code chunks

br.com.caelum.ichat.controller.PollingController

```java

@Controller
@RequestMapping("/polling")
public class PollingController {

	private BlockingQueue<Message> messages = new LinkedBlockingQueue<>();
	private Queue<DeferredResult<Message>> clients = new ConcurrentLinkedQueue<>();
	
	@PostConstruct
	public void init()  {
		new Thread(new Runnable() {
			@Override
			public void run() {
				while(true) {
					try {
                        			/* Retrieves and removes the head of this queue, waiting if necessary
						until an element becomes available.
                        
			                        If we want some timeout we can use 
        	        		        Message message = messages.pool(timeout, unitTime) */
						Message message = messages.take();

                			        /* When message is received it send to clients who are in the
			                        thread safe Queue clients waiting for some message. */
						sendToClients(message);
					} catch (Exception e) {
						e.printStackTrace();
					}
				}
			}
		}).start();
	}

	private void sendToClients(Message message)  {
		for (DeferredResult<Message> client : clients) {
			client.setResult(message);
		}
	}
	
	@ResponseBody
	@RequestMapping(method=RequestMethod.GET)
	public DeferredResult<Message> ouvirMensagem()  {
		
		/* Setup of client timeout and callback */
		long timeout = 20 * 1000L;
		final DeferredResult<Message> client = new DeferredResult<>(timeout);
		
		TimeoutCallback timeoutCallback = new TimeoutCallback(client, clients);
		ClientCallback clientCallback = new ClientCallback(client, clients);
		
		client.onTimeout(timeoutCallback);
		client.onCompletion(clientCallback);
		
		clients.offer(client);
		return client;
	}

	@RequestMapping(method=RequestMethod.POST)
	@ResponseStatus(HttpStatus.OK)
	public void doPost(@RequestBody Message message)  {
		messages.add(message);
	}

```
