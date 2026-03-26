Step 1: Extract and Import the project
Step 2: Open the terminal and run the following commands:
 
# 1. Install Python dependencies
pip install -r requirements.txt
 
# 2. Update .env file
update your GROQ_API_KEY
 
# 3.Backend dependencies
cd Backend
npm install
npm run dev
 
# 4.Frontend dependencies
cd frontend
npm install
npm run dev
 
 
 #5.Start Python server (Terminal 3)
 cd llm-eval-providers
python deepeval_server.py