---
layout: post
title: A Solution to Ruby Classes With Too Many Responsibilities
---

A while ago I was adding a stall detection feature to my LLM chat app. There were several places for the logic to go.

The first place I thought of was the Chat model. Essentially, I want to be able to do chat.is_stalling? which returns a boolean.

ruby
class Chat
...

  def is_stalling?
  ...
  end
end
But clearly, stall detection is not a core functionality of Chat. So stuffing it here is not good organization. What if we package it into a module or a concern?

ruby
class Chat
  include StallDetection

...
end

# In a concern/module
module StallDetection
  def is_stalling?
  ...
  end
end
This looks reasonable in my opinion. However, the module eventually grows as more granular methods for stall detection were added. And we see a problem:

ruby
module StallDetection
  def is_stalling?
  ...
  end

  private

  def punctuation_only_reply?
    ...
  end 

  def last_n_replies_short(...)?
    ...
  end

  def enough_conversation_history?
    ...
  end
  

  ...
end
It is very awkward that Chat has these private methods.

For example, punctuation_only_reply? would probably fit better in the Message model, but it is so peripheral to Message conceptually that I wouldn't put it there.

Adding a bunch of granular stall detection methods to Chat directly is conceptually awkward. So where do these behaviors belong? Another class.

It seems that there is an missing object that we can delegate to.

ruby
class Chat

  delegate :is_stalling?, to: :stall_detector
  
  def stall_detector
    @stall_detector ||= StallDetector.new(self)
  end

...
end

class StallDetector
  attr_reader :chat

  def initialize(...)
     ...
  end

  def is_stalling?
    ...
  end

  private

  ...
  

end
Now Chat does not have a bunch of peripheral methods, and is_stalling? is delegated to an object specialized for it. Note that StallDetector is not an ActiveRecord model, it is a plain Ruby class.
