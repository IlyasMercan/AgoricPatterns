# The seat structure diagram
<img src="https://raw.githubusercontent.com/IlyasMercan/AgoricPatterns/main/docs/legenda/images/legenda.PNG" width="600">

-   a

    -   Represents a non-creator called 'actor'.

    -   To achieve the goal of the smart contract, there should be n
        actor(s).

-   b

    -   Represents a creator, which is the entity that started the
        instance of the smart contract.

    -   There is exactly 1 creator.

-   c

    -   Represents an object called 'object' of type 'type'.

    -   There can be n instance(s) of 'object' for 1 instance of the
        smart contract.

-   d

    -   An object called 'object~1~' of type 'type~1~' can generate n~3~
        instance of an object called 'object~2~' of type 'type~2~'.

    -   There can be n~1~ instance(s) of 'object~1~' for 1 instance of
        the smart contract.

    -   There can be n~2~ instance(s) of 'object~2~' for 1 instance of
        the smart contract.

-   e

    -   An actor/creator can use 1 instance of an object called
        'object~1~' of type 'type~1~' to generate n~3~ instance of an
        object called 'object~2~' of type 'type~2~'.

    -   There can be n~1~ instance(s) of 'object~1~' for 1 instance of
        the smart contract.

    -   There can be n~2~ instance(s) of 'object~2~' for 1 instance of
        the smart contract.

-   f

    -   Seat 'seat~1~' and seat 'seat~2~' trade with one another.

    -   There can be n~1~ instance(s) of 'seat~1~' for 1 instance of the
        smart contract.

    -   There can be n~2~ instance(s) of 'seat~2~' for 1 instance of the
        smart contract.

-   g

    -   Seat seat~1~ and seat seat~2~ are equal to one another.

    -   There can be n~1~ instance(s) of 'seat~1~' for 1 instance of the
        smart contract.

    -   There can be n~2~ instance(s) of 'seat~2~' for 1 instance of the
        smart contract.

-   h

    -   The smart contract uses another smart contract called
        \"subcontract\", of which all related objects are positioned in
        the orange box.
