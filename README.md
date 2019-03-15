# PHP Design Pattern
The need for Design patterns in PHP can never be never be overemphasized. 
Whether you are building a small application or developing a very large application, using PHP Design pattern provides a robust way of developing more loose-coupled, readable, maintainable and well-structured code. 

There are many types of Design patterns however your choice depends what you plan to accomplish. When building a very large application you probably use different types of Design patterns in different parts of your applications.

Examples are always necessary for a better understanding of Design patterns so let’s get started with few examples:

## Chain-of-command Pattern.
```
<?php

/**
 * Interface CommandInterface
 * All commands must implement this interface
 */
interface CommandInterface{
    public function fail(Exception $exception, $name, array $args=[]);
}

/**
 * Class CommandException
 */
class CommandException extends Exception{}

/**
 * Class Command
 * Hold and run commands
 */
class Command{

    protected $commands = [];

    /**
     * Add command to list of commands
     *
     * @param $command
     * @return $this
     */
    public function addCommand($command){

        $command_name = $this->resolveCommandName($command);

        $this->commands[$command_name] = $command;

        return $this;
    }

    /**
     * Run command
     *
     * @param $name
     * @param array $args
     * @return mixed
     * @throws CommandException
     */
    public function runCommand($name, array $args=[]){

        //get command name
        $command_name = $this->getCommandName($name);
        $command_method = $this->getCommandMethod($name);

        //check if command exists
        if(!isset($this->commands[$command_name])){
            throw new CommandException("$command_name command does not exists");
        }

        //command
        $command =  $this->commands[$command_name];

        //check if method exists
        if(!method_exists($command, $command_method)){
            throw new CommandException("$command_method method does not exists in " .
                get_class($command));
        }


        //attempt to run command
        try{
            return call_user_func_array([$command, $command_method], $args);
        }catch(Exception $e){
            //we call failure handler of command
            $command->fail($e, $name, $args);
        }
    }

    /**
     * Resolve command name from command class name
     *
     * @param $command
     * @return string
     */
    protected function resolveCommandName($command){
        return str_replace('command', '', strtolower(get_class($command)));
    }

    /**
     * Return command name from command:method string
     * @param $name
     * @return string
     */
    protected function getCommandName($name){
        return explode(':', $name)[0];
    }

    /**
     * Return method name from command:method string
     * @param $name
     * @return string
     */
    protected function getCommandMethod($name){
        return explode(':', $name)[1];
    }
}

class UserCommand implements CommandInterface{

    /**
     * Handle failure
     * @param Exception $exception
     * @param $name
     * @param array $args
     */
    public function fail(Exception $exception, $name, array $args=[]){
        echo $exception->getMessage() . "\n";
    }

    /**
     * Register user
     *
     * @param $name
     * @param $email
     * @return string
     */
    public function register($name, $email){
        //registration code
        echo "$name just registered, Email: $email\n";
    }
}

class MailCommand implements CommandInterface{

    /**
     * Handle failure
     *
     * @param Exception $exception
     * @param $name
     * @param array $args
     */
    public function fail(Exception $exception, $name, array $args=[]){
        //handle what happens on command failure
        echo $exception->getMessage() . "\n";
    }

    /**
     * Send email
     *
     * @param $email
     * @param $message
     */
    public function send($email, $message){
        //registration code
        echo "Email sent to : $email, Message: $message \n";
    }
}

$command = new Command();

$command->addCommand(new UserCommand());
$command->addCommand(new MailCommand());

$command->runCommand('user:register', ['name' => 'Emeka', 'email' => 'john.doe@email.com']);
//Emeka just registered, Email: john.doe@email.com

$command->runCommand('mail:send', ['email' => 'john.doe@email.com', 'message' => 'Thank You :)']);
//Email sent to : john.doe@email.com, Message: Thank You :)
```
