# secure-s3-cloudfront-site

## Overview
Host a static HTML website on Amazon S3, served securely through a
CloudFront CDN distribution with HTTPS, DDoS protection, and a custom domain.

## Architecture
User visits https://digitalculture.com
  → Route 53 resolves DNS via A record (Alias) pointing to CloudFront
  → CloudFront receives the request (DDoS protected by AWS Shield Standard)
      → Cache HIT  → serves content instantly from edge location
      → Cache MISS → fetches from origin (S3 static website endpoint)
  → S3 returns the static HTML/CSS/JS files

## Technologies Used
- Amazon S3              — static file storage and website hosting
- Amazon CloudFront      — CDN, HTTPS termination, edge caching
- Amazon Route 53        — DNS management and custom domain routing
- AWS Certificate Manager (ACM) — free SSL/TLS certificate
- AWS Shield Standard    — automatic DDoS protection (included with CloudFront)

## Deployment Steps

  1. Create S3 bucket
       - Name: digitalculture.com
       - Enable static website hosting (index.html / 404.html)
       - Uncheck "Block all public access"
       - Attach custom cloudfront policy

  2. Request SSL certificate in ACM
       - Region: us-east-1 (required for CloudFront)
       - Domains: digitalculture.com and *.digitalculture.com
       - Validate via Route 53 (auto-creates CNAME record)

  3. Create CloudFront distribution
       - Origin: S3 website endpoint
       - Redirect HTTP → HTTPS
       - Attach ACM certificate
       - Set default root object: index.html

  4. Configure Route 53
       - A record (Alias) for digitalculture.com → CloudFront distribution
       - A record (Alias) for www.digitalculture.com → same distribution

  5. Upload files to S3
       - Upload HTML/CSS/JS files via AWS Console or CLI
       - aws s3 sync ./your-site s3://digitalculture.com --delete

  6. Invalidate CloudFront cache (after any update)
       - aws cloudfront create-invalidation \
           --distribution-id <YOUR_DIST_ID> --paths "/*"

## Key Learnings
- ACM certificates must be created in us-east-1 regardless of where
  your S3 bucket lives — CloudFront only reads certs from that region
- Use the S3 *website endpoint* as the CloudFront origin, not the
  REST endpoint — otherwise index.html won't serve for subdirectories
- CloudFront cache doesn't auto-clear on S3 upload; you must run a
  cache invalidation (/*) after every update or visitors see stale content
- Route 53 A records for CloudFront must use Alias, not a plain CNAME —
  Alias is free and works on the apex domain (naked domain) where CNAMEs are not allowed
- AWS Shield Standard is automatically enabled on all CloudFront
  distributions at no extra cost
