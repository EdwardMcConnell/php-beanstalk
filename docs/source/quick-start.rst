Quickstart
----------

As a Producer
*************

.. sourcecode:: php

    // returns BeanstalkPool instance
    $bean = Beanstalk::init();
    $bean->addServer('localhost', 11300)
         ->useTube('my-tube')
         ->put('Hello World!');

As a Consumer
*************

.. sourcecode:: php

    $bean = Beanstalk::init();
    $bean->addServer('localhost', 11300)
         ->watchTube('my-tube');

    while (true)
    {
        try
        {
            $job = $bean->reserve($timeout = 10);

            /* process job ... */

            $job->delete();
        }
        catch (BeanstalkException $e)
        {
            switch ($e->getCode())
            {
                case BeanstalkException::TIMED_OUT:
                    echo "Timed out waiting for a job.  Retrying in 1 second."
                    sleep(1);
                    continue;
                    break;

                default:
                    throw $e;
                    break;
            }
        }
    }

Built in JSON support
*********************

Objects are automatically converted
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. sourcecode:: php

    $bean = Beanstalk::init();
    $bean->addServer('localhost', 11300)
         ->useTube('my-tube');

    $obj = new stdClass;
    $obj->content = 'Hello World!';
    $bean->put($obj); // stored in beanstalkd as '{"content":"Hello World!"}'

    $bean->watchTube('my-tube');
    $job = $bean->reserve();
    print_r($job->getMessage());

Outputs::

    stdClass Object
    (
        [content] => Hello World!
    )

Send a custom JSON string
^^^^^^^^^^^^^^^^^^^^^^^^^

.. sourcecode:: php

    $bean = Beanstalk::init();
    $bean->addServer('localhost', 11300)
         ->useTube('my-tube')
         ->put('[123,456,789]');

    $bean->watchTube('my-tube');
    $job = $bean->reserve();
    print_r($job->getMessage());

Outputs::

    Array
    (
        [0] => 123
        [1] => 456
        [2] => 789
    )
