# Event

## 冒泡和捕获

event.target
This property of event objects is the object the event was dispatched on. It is different than event.currentTarget when the event handler is called in bubbling or capturing phase of the event.

event.currentTarget
Identifies the current target for the event, as the event traverses the DOM. It always refers to the element the event handler has been attached to as opposed to event.target which identifies the element on which the event occurred.

event.target指向引起触发事件的元素(捕获阶段)，而event.currentTarget则是事件绑定的元素（冒泡阶段），只有被点击的那个目标元素的event.target才会等于event.currentTarget