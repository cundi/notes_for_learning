# 获取文章评论数的爬虫

引用：`https://medium.com/python-pandemonium/asyncio-coroutine-patterns-beyond-await-a6121486656f`

```python
"""
这里我们那想要递归地取得特定文章的子评论数量。
"""

import asyncio
import argparse
import logging
from urllib.parse import urlparse, parse_qs
from datetime import datetime

import aiohttp
import async_timeout


LOGGER_FORMAT = '%(asctime)s %(message)s'
URL_TEMPLATE = "https://hacker-news.firebaseio.com/v0/item/{}.json"
FETCH_TIMEOUT = 10

parser = argparse.ArgumentParser(
    description='计算黑客新闻文章')
parser.add_argument('--id', type=int, default=8863,
                    help='黑客新闻中的文章ID，默认位8863')
parser.add_argument('--url', type=str, help='文章的URL')
parser.add_argument('--verbose', action='store_true', help='详情输出')


logging.basicConfig(format=LOGGER_FORMAT, datefmt='[%H:%M:%S]')
log = logging.getLogger()
log.setLevel(logging.INFO)

fetch_counter = 0


async def fetch(session, url):
    """获取URL后使用aiohttp返回解析过的JSON响应，
    根据aiohttp文档的建议这里我们复用session。
    """
    global fetch_counter
    with async_timeout.timeout(FETCH_TIMEOUT):
        fetch_counter += 1
        async with session.get(url) as response:
            return await response.json()


async def post_number_of_comments(loop, session, post_id):
    """重新取回当前文章的数据，并取得全部评论。
    """
    url = URL_TEMPLATE.format(post_id)
    now = datetime.now()
    response = await fetch(session, url)
    log.debug('{:^6} > Fetching of {} took {} seconds'.format(
        post_id, url, (datetime.now() - now).total_seconds()))

    if 'kids' not in response:  # base case, there are no comments
        return 0

    # calculate this post's comments as number of comments
    number_of_comments = len(response['kids'])

    # 对全部的评论创建task
    log.debug('{:^6} > Fetching {} child posts'.format(
        post_id, number_of_comments))
    tasks = [post_number_of_comments(
        loop, session, kid_id) for kid_id in response['kids']]

    # 计划执行task，并取回结果
    results = await asyncio.gather(*tasks)

    # 计算子评论总和，并与文章关联
    number_of_comments += sum(results)
    log.debug('{:^6} > {} comments'.format(post_id, number_of_comments))

    return number_of_comments


def id_from_HN_url(url):
    """返回URL的id值，否则None
    """
    parse_result = urlparse(url)
    try:
        return parse_qs(parse_result.query)['id'][0]
    except (KeyError, IndexError):
        return None


async def main(loop, post_id):
    """协程入口
    """
    now = datetime.now()
    async with aiohttp.ClientSession(loop=loop) as session:
        now = datetime.now()
        comments = await post_number_of_comments(loop, session, post_id)
        log.info(
            '> 计算评论耗时 {:.2f} 秒 ， 取得 {}'.format(
                (datetime.now() - now).total_seconds(), fetch_counter))

    return comments


if __name__ == '__main__':
    args = parser.parse_args()
    if args.verbose:
        log.setLevel(logging.DEBUG)

    post_id = id_from_HN_url(args.url) if args.url else args.id

    loop = asyncio.get_event_loop()
    comments = loop.run_until_complete(main(loop, post_id))
    log.info("-- 文章 {} 包含 {} 评论".format(post_id, comments))

    loop.close()
```

asyncio.gather等待所有的协程都完成后，再返回它们的结果。

## 撒手不管（fire and forget）

