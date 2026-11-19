# Lab 09: Knowledge check

**Answer each question below it, in your own words, in one to three sentences.** Replace the `...` below **Your answer:** with your text.

## 1. Credentials in the image?

Assume you want to share your image with others and make it publicly available on `ghcr.io`, so that anyone can pull it and run the analysis. A teammate suggests adding `COPY .aws /home/app/.aws` to the `Dockerfile`, so that nobody has to type `-v ~/.aws:/home/app/.aws:ro` any more. (This is a thought experiment: keep your own package private.)

**Your answer:** Give two reasons why this is a bad idea. Think of who could then read your credentials, and of what happens when your Learner Lab session ends.

This is a bad idea because it would allow people to have access to your aws credentials which could allow them to see your keys and have the ability to
use your cloud resources or steal your data, so it's a security risk. It's also a bad idea because when the Learner Lab session ends, the images will be
broken because the credentials will no longer be valid, so you would need to completely rebuild the images with new credentials.

## 2. Where did the Atlas password come from?

In Step 4, you ran the analysis once with `-v ~/.aws:/home/app/.aws:ro` only, without the three `-e` options. The container still wrote to your Atlas cluster.

*Hint:* Read `get_atlas_settings()` in `src/book_analysis/analyze.py`.

**Your answer:** Explain, step by step, where the container found the Atlas settings in that run.

The get_atlas_settings() method checks if "MONGODB_ATLAS_URL" is in os.environ, but because the -e environment variables were not passed 
into the container, "MONGODB_ATLAS_URL" is not set in os.environ. So then a boto3 SSM client is created for REGION and calls 
get_parameter(Name=SSM_PARAMETER, WithDecryption=True). To authenticate the boto3 SSM API call, boto3 looks for AWS credentials in the ~/.aws directory.
The AWS SSM then returns the encrypted Atlas settings, boto3 decrypts the settlings, and json.loads(resp["Parameter"]["Value"]) reads the JSON so
the container gets the Atlas settings it needs.

## 3. Same tag, different image

Run `docker images book-analysis`, and note the image ID of `book-analysis:0.1.0`. Then change the `description` in the `LABEL` of your `Dockerfile`, build again with the same command, `docker build -t book-analysis:0.1.0 .`, and run `docker images book-analysis` again.

**Your answer:** Paste the image ID from before and after the rebuild. Why is it a problem for someone who uses your image that `0.1.0` now names a different image, and what should you have done instead?

Image ID before rebuild: c0ae25441b10

Image ID after rebuild: 32a7435bc19f

It is a problem because anyone who runs book-analysis:0.1.0 will get a different image than before, so someone who used the image before won't get the
same image which could cause problems for what they're trying to do. Instead, I should have changed the version tag to something like 0.2.0 so that 0.1.0
will still point to the original image and 0.2.0 points to the new image, and people can differentiate the two. 
