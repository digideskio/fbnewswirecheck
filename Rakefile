require 'dotenv/tasks'
require 'postmark'

task default: %w[check]

task :check => :dotenv do
  require 'time'
  require 'koala'
  
  WARN_TIME = Integer(ENV['WARN_TIME'])  # in minutes
  WARN_RECIPIENTS = 'paul.watson+warn@storyful.com' # Email recipients to email on warning
  ERROR_RECIPIENTS = 'paul.watson+error@storyful.com' # Email recipients to email on error
  INFO_RECIPIENTS = 'paul.watson+info@storyful.com' # Email recipients to email on info
  FROM_EMAIL = 'sysadmin@storyful.com'
  
  
  begin
    graph = Koala::Facebook::API.new(ENV['FACEBOOK_GRAPH_API_TOKEN'])
    
    client = Postmark::ApiClient.new(ENV['POSTMARK_API_KEY'])
  
    posts = graph.get_object("fbnewswire/feed?limit=1")
    if posts.length > 0
      latest_post_time = Time.parse(posts[0]['created_time'])
      time_diff = ((Time.now - latest_post_time) / 60).round
      if time_diff >= WARN_TIME
        puts "INFO: WARN: No post in the last #{WARN_TIME} minutes"
        client.deliver( from: FROM_EMAIL,
                        to: WARN_RECIPIENTS,
                        subject: "[FBNEWSWIRE] No post in the last #{WARN_TIME} minutes",
                        text_body: "There was no new post on facebook.com/fbnewswire in the past #{WARN_TIME} minutes.")
      else
        puts "INFO: OK: Post in the last #{WARN_TIME} minutes"
        client.deliver( from: FROM_EMAIL,
                        to: INFO_RECIPIENTS,
                        subject: "[FBNEWSWIRE] Post in the last #{WARN_TIME} minutes",
                        text_body: "There was a new post on facebook.com/fbnewswire in the past #{WARN_TIME} minutes.")
      end
    else
      puts 'ERROR: No posts'
      client.deliver( from: FROM_EMAIL,
                      to: ERROR_RECIPIENTS,
                      subject: "[FBNEWSWIRE] ERROR, NO POSTS",
                      text_body: "No posts returned")
    end
  rescue Exception => e  
    puts "EXCEPTION: #{e.message}"
    client.deliver( from: FROM_EMAIL,
                    to: ERROR_RECIPIENTS,
                    subject: "[FBNEWSWIRE] EXCEPTION",
                    text_body: "#{e.message}")
  end
end