  DemoSteps                                                                                                                                                           
                                                                                                                                                                       
  Step 1 — Make sure your EC2 container is running                                                                                                                     
  ssh -i /Users/mac/Downloads/springboot-demo.pem ubuntu@34.221.51.27 "sudo docker ps"                                                                                 
  You should see the sms container with status Up.                                                                                                                     
                                                                                                                                                                       
  Step 2 — Trigger the endpoint                                                                                                                                        
  curl -s -X POST http://34.221.51.27:8080/name/aggregation | python3 -m json.tool                                                                                     
  You'll immediately get back:                 


  You'll immediately get back:                                                                                                                                         
  {                                                                                                                                                                    
      "name": ["Jessica"]                                                                                                                                              
  }                                                                                                                                                                    
                                                                                                                                                                       
  Step 3 — Check the logs to confirm the async forward fired                                                                                                           
  ssh -i /Users/mac/Downloads/springboot-demo.pem ubuntu@34.221.51.27 "sudo docker logs sms 2>&1 | tail -10"                                                           
  - If Joycelin's service is up → you'll see a success log with her response                                                                                           
  - If Joycelin's service is down → you'll see Failed to forward to next service — that's fine, your part still worked                                                 
                                                                                                                                                                       
  Step 4 — Show the class the chain in action                                                                                                                          
                                                                                                                                                                       
  Once everyone has deployed, trigger Step 2 again and check the logs. Joycelin will have received {"name": ["Jessica"]}, added her name, and forwarded to the next    
  person — all the way down the chain.                                                                                                                                 
                                        
                                                                                                                    
                                           