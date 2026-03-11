## Understanding the Architecture

```
User Browser
    ↓ HTTPS
CloudFront (CDN)
    ↓ 
S3 Static Website (Frontend)
    ↓ HTTPS API Calls
API Gateway
    ↓
Lambda Function (Backend)
    ↓
    ├── AWS Bedrock (AI responses)
    └── S3 Memory Bucket (persistence)
```

### Key Components

1. **CloudFront**: Global CDN, provides HTTPS, caches static content
2. **S3 Frontend Bucket**: Hosts static Next.js files
3. **API Gateway**: Manages API routes, handles CORS
4. **Lambda**: Runs your Python backend serverlessly
5. **S3 Memory Bucket**: Stores conversation history as JSON files

## Cost Optimization Tips

### Current Costs (Approximate)

- Lambda: First 1M requests free, then $0.20 per 1M requests
- API Gateway: First 1M requests free, then $1.00 per 1M requests  
- S3: ~$0.023 per GB stored, ~$0.0004 per 1,000 requests
- CloudFront: First 1TB free, then ~$0.085 per GB
- **Total**: Should stay under $5/month for moderate usage

### How to Minimize Costs

1. **Use CloudFront caching** - reduces requests to origin
2. **Set appropriate Lambda timeout** - don't set unnecessarily high
3. **Monitor with CloudWatch** - set up billing alerts
4. **Clean old S3 files** - delete old conversation logs periodically
5. **Use AWS Free Tier** - many services have generous free tiers