const express = require('express');
const Parser = require('rss-parser');
const cors = require('cors');

const app = express();
const parser = new Parser();
app.use(cors());

const RSS_FEEDS = [
  'https://rss.nytimes.com/services/xml/rss/nyt/HomePage.xml',
  'https://feeds.bbci.co.uk/news/rss.xml',
  'https://hnrss.org/frontpage'
];

app.get('/news', async (req, res) => {
  try {
    let allArticles = [];

    for (const feed of RSS_FEEDS) {
      const parsed = await parser.parseURL(feed);
      allArticles.push(...parsed.items.map(item => ({
        title: item.title,
        link: item.link,
        pubDate: item.pubDate,
        source: parsed.title
      })));
    }

    allArticles.sort((a, b) => new Date(b.pubDate) - new Date(a.pubDate));
    res.json(allArticles);
  } catch (err) {
    res.status(500).json({ error: 'Failed to fetch news' });
  }
});

const PORT = process.env.PORT || 4000;
app.listen(PORT, () => console.log(`Backend running on port ${PORT}`));
