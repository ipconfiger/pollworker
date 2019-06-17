# pollworker
Polling queue and distribute messages to worker processes and process it with your own function

### Installation

    sudo pip install pollworker
    
### Usage

    import pollworker

#### Step1: Define master process

Define a class contains method named poll, with no parameter

    import redis
    class RedisPoller(object):
        def __init__(self, host, port):
            self.redis = redis.StrictRedis(host=host, port=port)
        
        def poll():
            recieved = self.redis.brpop('beat', timeout=120)
            if recieved:
                chn, message = recieved
                return message
    
If use <lpush beat "some message"> to redis, Poller will distribute the message to 
worker process.

#### Step2: Define worker process

Define a function with two parameter, the first one will get the id of the worker process, the second parameter will get the message from master process.

    def worker(pid, message):
        logging.info('worker(%s) got message:%s', pid, message)
        
#### Step3: Last step - regist worker, master and start

    pollworker.regist_worker(worker)
    pollworker.regist_poller(RedisPoller('127.0.0.1', 6379))
    pollworker.start()

The start function has two options

1. workers: Directly set the number of worker process, if leave it blank or set to 0, proccess count depends on your server's CPU count.
2. stopwaits: Seconds master process will wait for subprocess on SIGTERM.       



