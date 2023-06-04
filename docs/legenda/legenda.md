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

    -   An object called 'object<sub>1</sub>' of type 'type<sub>1</sub>' can generate n<sub>3</sub> instance of an object called 'object<sub>2</sub>' of type 'type<sub>2</sub>'.

    -   There can be n<sub>1</sub> instance(s) of 'object<sub>1</sub>' for 1 instance of
        the smart contract.

    -   There can be n<sub>2</sub> instance(s) of 'object<sub>2</sub>' for 1 instance of
        the smart contract.

-   e

    -   An actor/creator can use 1 instance of an object called
        'object<sub>1</sub>' of type 'type<sub>1</sub>' to generate n<sub>3</sub> instance of an
        object called 'object<sub>2</sub>' of type 'type<sub>2</sub>'.

    -   There can be n<sub>1</sub> instance(s) of 'object<sub>1</sub>' for 1 instance of
        the smart contract.

    -   There can be n<sub>2</sub> instance(s) of 'object<sub>2</sub>' for 1 instance of
        the smart contract.

-   f

    -   Seat 'seat<sub>1</sub>' and seat 'seat<sub>2</sub>' trade with one another.

    -   There can be n<sub>1</sub> instance(s) of 'seat<sub>1</sub>' for 1 instance of the
        smart contract.

    -   There can be n<sub>2</sub> instance(s) of 'seat<sub>2</sub>' for 1 instance of the
        smart contract.

-   g

    -   Seat seat<sub>1</sub> and seat seat<sub>2</sub> are equal to one another.

    -   There can be n<sub>1</sub> instance(s) of 'seat<sub>1</sub>' for 1 instance of the
        smart contract.

    -   There can be n<sub>2</sub> instance(s) of 'seat<sub>2</sub>' for 1 instance of the
        smart contract.

-   h

    -   The smart contract uses another smart contract called
        \"subcontract\", of which all related objects are positioned in
        the orange box.
