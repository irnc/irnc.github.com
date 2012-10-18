# How Derby parses x-bind value?

See splitEvents() function in eventBindings.js

value = eventBinding[, value]
eventBinding = eventName[/delay]: callbacks
callbacks = callback[ callbacks]
