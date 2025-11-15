# Fitsol ESG Platform

An intelligent ESG (Environmental, Social, and Governance) analytics platform that leverages AI agents to provide carbon accounting, benchmarking, and net-zero planning capabilities.

## 🌟 Features

- **🤖 Multi-Agent System**: Specialized AI agents for different ESG tasks
- **🧠 Intelligent RAG**: Context-aware responses using knowledge base
- **⚡ Real-time Processing**: Asynchronous task execution and coordination
- **📊 Carbon Accounting**: Calculate and track carbon emissions (Scope 1, 2, 3)
- **📈 Benchmarking**: Compare ESG metrics against industry standards
- **🎯 Net Zero Planning**: Create and validate net-zero pathways
- **📁 File Processing**: Support for CSV, PDF, Excel, and more
- **📉 Visualization**: Interactive charts and graphs for ESG metrics
- **👥 User Management**: Role-based access control (RBAC)
- **🔒 Secure**: JWT authentication and authorization

## 🏗️ Architecture

The system is built on a modern microservices-oriented architecture:

- **Frontend**: React 19 + TypeScript + Vite
- **Backend**: FastAPI + Python 3.10+
- **Database**: MongoDB (primary) + SQLite (reference data)
- **AI/ML**: LangChain + Google Gemini API
- **Deployment**: Docker + Docker Compose

For detailed architecture documentation, see [ARCHITECTURE.md](./ARCHITECTURE.md)

For agent system documentation, see [AGENTS.md](./AGENTS.md)

## 🚀 Quick Start

### Prerequisites

- Python 3.10 or higher
- Node.js 18 or higher
- MongoDB 4.4 or higher (or MongoDB Atlas connection string)
- Google Gemini API key (or OpenAI/Anthropic API key)

### Installation

1. **Clone the repository:**
```bash
git clone <repository-url>
cd fitsol_ant
```

2. **Set up backend:**
```bash
cd server
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

3. **Set up frontend:**
```bash
cd client
npm install
```

4. **Configure environment:**
```bash
# Copy example environment file
cp env.example .env

# Edit .env with your configuration
# Required:
# - MONGODB_CONNECTION_STRING
# - GEMINI_API_KEY (or OPENAI_API_KEY/ANTHROPIC_API_KEY)
# - JWT_SECRET_KEY
```

### Running the Application

**Development Mode:**

```bash
# Terminal 1: Start backend
cd server
./run_backend.sh
# Backend will run on http://localhost:8000

# Terminal 2: Start frontend
cd client
npm run dev
# Frontend will run on http://localhost:5173
```

**Docker (Recommended for Production):**

```bash
cd deployment
docker-compose up -d
# Application will be available on http://localhost
```

## 📖 Documentation

- **[ARCHITECTURE.md](./ARCHITECTURE.md)**: Complete technical architecture documentation
- **[AGENTS.md](./AGENTS.md)**: Detailed agent system documentation
- **[server/config/CONFIGURATION_GUIDE.md](./server/config/CONFIGURATION_GUIDE.md)**: Configuration guide
- **[server/utils/FILE_PROCESSING_GUIDE.md](./server/utils/FILE_PROCESSING_GUIDE.md)**: File processing guide

## 🎯 Usage Examples

### Carbon Accounting

```python
# Calculate Scope 1 emissions
POST /carbon/calculate
{
  "query": "Calculate Scope 1 emissions for 1000 liters of diesel fuel"
}

# Response
{
  "result": {
    "total_emissions": 2.68,
    "unit": "tCO2e",
    "breakdown": {
      "scope1": 2.68,
      "scope2": 0.0,
      "scope3": 0.0
    }
  }
}
```

### Benchmarking

```python
# Compare carbon intensity
POST /benchmarking/analyze
{
  "query": "Compare our carbon intensity to industry average",
  "metric": "carbon_intensity",
  "sector": "manufacturing"
}
```

### Net Zero Planning

```python
# Create net zero plan
POST /netzero/create-plan
{
  "query": "Create a net zero plan for 2050",
  "baseline_year": 2024,
  "target_year": 2050
}
```

## 🧪 Testing

```bash
# Run all tests
pytest

# Run unit tests
pytest tests/unit/

# Run integration tests
pytest tests/integration/

# Run agent tests
pytest tests/ml/agents/

# With coverage
pytest --cov=server --cov-report=html
```

## 🔧 Configuration

### Environment Variables

Key environment variables (see `env.example` for full list):

```bash
# Database
MONGODB_CONNECTION_STRING=mongodb://localhost:27017/fitsol_esg
MONGODB_DB_NAME=fitsol_esg

# LLM Provider
GEMINI_API_KEY=your_api_key_here
LLM_PROVIDER=gemini  # Options: gemini, openai, anthropic
LLM_MODEL=gemini-2.0-flash-001

# Authentication
JWT_SECRET_KEY=your-secret-key-change-in-production
JWT_ACCESS_TOKEN_EXPIRE_HOURS=24

# Feature Flags
ENABLE_SUPERVISOR_AGENT=true
USE_INTELLIGENT_CLARIFICATION=true
ENABLE_VISUALIZATION=true
```

For detailed configuration options, see [CONFIGURATION_GUIDE.md](./server/config/CONFIGURATION_GUIDE.md)

## 📁 Project Structure

```
fitsol_ant/
├── client/                 # React frontend
│   ├── src/
│   │   ├── components/    # React components
│   │   ├── pages/         # Page components
│   │   ├── services/      # API services
│   │   └── hooks/         # Custom hooks
│   └── package.json
├── server/                 # FastAPI backend
│   ├── api/               # API routes
│   ├── ml/                # ML/AI components
│   │   └── agents/       # Agent system
│   ├── database/          # Database models
│   ├── config/            # Configuration
│   └── requirements.txt
├── deployment/             # Docker deployment files
├── tests/                  # Test suite
├── ARCHITECTURE.md         # Architecture documentation
├── AGENTS.md              # Agent documentation
└── README.md              # This file
```

## 🔐 Security

- **Authentication**: JWT-based authentication
- **Authorization**: Role-based access control (RBAC)
- **Password Hashing**: bcrypt
- **CORS**: Configurable CORS policy
- **Input Validation**: Pydantic models for request validation
- **SQL Injection Protection**: Parameterized queries
- **XSS Protection**: Input sanitization

## 🚀 Deployment

### Docker Deployment

```bash
cd deployment
docker-compose up -d
```

### Production Checklist

- [ ] Set strong `JWT_SECRET_KEY`
- [ ] Configure MongoDB with authentication
- [ ] Set up SSL/TLS certificates
- [ ] Configure CORS for production domains
- [ ] Set up monitoring and logging
- [ ] Configure backup strategy
- [ ] Set up health checks
- [ ] Configure rate limiting

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

[Add your license information here]

## 🆘 Support

For issues and questions:
- Open an issue on GitHub
- Check the documentation in `ARCHITECTURE.md` and `AGENTS.md`
- Review the configuration guide

## 🗺️ Roadmap

- [ ] Microservices architecture migration
- [ ] Redis caching layer
- [ ] WebSocket support for real-time updates
- [ ] GraphQL API
- [ ] Mobile app (React Native)
- [ ] Advanced analytics dashboard
- [ ] Multi-tenant support
- [ ] API rate limiting
- [ ] Enhanced monitoring and observability

## 🙏 Acknowledgments

- Built with [FastAPI](https://fastapi.tiangolo.com/)
- Powered by [React](https://react.dev/)
- AI capabilities via [Google Gemini](https://ai.google.dev/)
- Vector search with [FAISS](https://github.com/facebookresearch/faiss)

---

**Made with ❤️ by the Fitsol Team**

