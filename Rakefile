require 'postmark'
require 'time'
require 'koala'

WARN_TIME = Integer(ENV['WARN_TIME'])  # in minutes
WARN_RECIPIENTS = ENV['WARN_RECIPIENTS'] # Email recipients to email on warning
ERROR_RECIPIENTS = ENV['ERROR_RECIPIENTS'] # Email recipients to email on error
INFO_RECIPIENTS = ENV['INFO_RECIPIENTS'] # Email recipients to email on info
FROM_EMAIL = ENV['FROM_EMAIL']

task default: %w[check]

task :check do
  begin
    graph = Koala::Facebook::API.new(ENV['FACEBOOK_GRAPH_API_TOKEN'])
    
    client = Postmark::ApiClient.new(ENV['POSTMARK_API_KEY'])
  
    posts = graph.get_object("fbnewswire/feed")
    if posts.length > 0
      start_time = Time.now
      smallest_diff = WARN_TIME
      
      posts.each do |post|
        post_time = Time.parse(post['created_time'])
        time_diff = ((start_time - post_time) / 60).round
        puts time_diff
        puts post_time
        
        if (time_diff < smallest_diff)
          smallest_diff = time_diff
        end
      end
        
      if smallest_diff >= WARN_TIME
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

task :get_token_url do
  oauth = Koala::Facebook::OAuth.new(ENV['FACEBOOK_APP_ID'], ENV['FACEBOOK_APP_SECRET'], 'http://fbnewswirecheck.dev/')
  puts "INFO: URL for oAuth Code: #{oauth.url_for_oauth_code(permissions: 'read_stream')}"
  puts "INFO: Follow the URL, authorise then copy code key from address bar."
  puts "INFO: Now run rake refresh_token code=THECODEHERE"
end

task :refresh_token do
  puts "INFO: Now visit: https://graph.facebook.com/oauth/access_token?client_id=#{ENV['FACEBOOK_APP_ID']}&redirect_uri=http://fbnewswirecheck.dev/&client_secret=#{ENV['FACEBOOK_APP_SECRET']}&code=#{ENV['code']}"
  puts "INFO: And copy the access token and set FACEBOOK_GRAPH_API_TOKEN value."
end