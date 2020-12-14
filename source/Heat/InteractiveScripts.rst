===============================
Automating Interactive Scripts
===============================

Sometimes there are installations which require user responses to certain questions.
One example of this can be found when running the mysql_secure_installation command.
It is possible to automate these responses when you want to have a database, like MySQL
set up and ready to use as root.

To do this, we use **expect**.

According to the linux man page, expect is programmed dialogue which knows what is expected from the program and what the
correct response should be.

Typically, Expcect is run in a separate script with the first line of the script as:

.. code-block:: bash

  #!/usr/local/bin/expect -<options>


However, it is possible to run an expect script within a regular bash script.

An expect script inside a normal bash script would be written as follows:

.. code-block:: bash
  $FOO=(expect -c "

  expect \"<interactive prompt>\"

  send \"<response>\r\"

  ")
  echo "$FOO"

References
##########

https://linux.die.net/man/1/expect

https://gist.github.com/Mins/4602864
