# Week 8 — Portfolio theory & allocation

> **Goal:** by Sunday you can take a handful of assets' historical prices, turn them into a covariance matrix in SQL and pandas, trace the entire efficient frontier, find the maximum-Sharpe and minimum-variance portfolios, and defend the resulting weights against a skeptical CFO who asks "why *this* mix and not equal weights?"

Welcome back to **C35 · Crunch Capital**. Every week so far has looked at value from the perspective of a single asset or a single deal — one company's WACC, one project's NPV, one financing round's dilution. This week the unit of analysis changes: instead of "is this one thing good," the question becomes "what is the best **combination** of things," and the answer turns out to depend on something most beginners underweight — not just how risky each asset is on its own, but how those assets move *together*. That single idea, formalized by Harry Markowitz in 1952 and still the backbone of every institutional asset-allocation process today, is **modern portfolio theory (MPT)**: combining imperfectly correlated assets can produce a portfolio with a better return-per-unit-of-risk than any single asset in it — a rare case in finance where you get something close to a free lunch.

This week we build a six-fund investment universe for a fictional multi-asset shop, **Crunch Capital Advisors**: a US equity fund, an international equity fund, a core bond fund, a high-yield bond fund, a gold trust, and a real estate fund — five years of realistic monthly prices for all six, plus a market benchmark for the CAPM work in Lecture 3. You'll compute returns, variances, and the full covariance matrix by hand and in pandas; use mean-variance optimization to trace the efficient frontier; solve for the global-minimum-variance and maximum-Sharpe portfolios in closed form; and finish by pricing each fund's CAPM-required return and reasoning about factor tilts. By Sunday, "diversification reduces risk" stops being a slogan and becomes a number you can compute and defend.

## Learning objectives

By the end of this week, you will be able to:

- **Compute** asset returns, variance, covariance, and correlation from raw price history in SQL and pandas — and explain *why* covariance, not just individual volatility, is the quantity that determines a portfolio's risk.
- **Build** a full covariance matrix for a multi-asset universe and use it to compute the expected return and volatility of an arbitrary set of portfolio weights.
- **Trace** the efficient frontier with mean-variance optimization — the set of portfolios that deliver the highest expected return for every level of risk.
- **Solve** in closed form for the global-minimum-variance portfolio and the maximum-Sharpe (tangency) portfolio, and explain what distinguishes them.
- **Apply** CAPM to price each asset's required return from a regression beta, and reason honestly about adding factor tilts (over/underweighting by beta, size, or another characteristic) to a baseline allocation.
- **Reason honestly** about the limits of naive mean-variance optimization — estimation error, unstable weights, extreme long/short positions, and why "optimal" on paper is not the same as "investable" in practice.

## Prerequisites

- **Week 4 of this course** (cost of capital & WACC) — you already computed a regression beta from price data and applied CAPM to get one company's cost of equity; this week reuses that exact machinery (`pct_change`, `numpy.polyfit` or `numpy.cov`) across six assets at once instead of one.
- Your Week 1 workspace: PostgreSQL 16+ (or SQLite 3.35+ fallback) and a Python environment with `pandas`, `numpy`, and `sqlalchemy` installed.
- Comfort with basic matrix/vector notation (a portfolio's variance is a **quadratic form**, `w' Σ w` — if that phrase is unfamiliar, Lecture 1 rebuilds it from scratch with plain arithmetic first).
- Working SQL (`SELECT`, `WHERE`, `GROUP BY`, joins) from Week 1 of this course, or [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) if you need a refresher.
- **No spreadsheets**, per the [course's data tooling rule](../../SYLLABUS.md#data-tooling-rule). Every price series and every weight vector this week lives in SQL and pandas — a portfolio optimizer built in a spreadsheet is exactly the kind of workflow this course exists to replace, because a `numpy.linalg.inv()` call is auditable, reproducible, and doesn't silently break when someone inserts a row.

## Set up this week's data (do this first)

You need the PostgreSQL 16+ (or SQLite 3.35+) database from Week 1, plus your Python environment with `pandas`, `numpy`, and `sqlalchemy`. This week adds two new tables to the same `crunch_capital` database: `asset_prices` and `capital_market_assumptions`.

```bash
psql crunch_capital              # PostgreSQL
# or
sqlite3 crunch_capital.db        # SQLite fallback
```

Paste this into your shell — it works unchanged on both engines:

```sql
-- 1. Five years of monthly closing prices (Dec 2020 - Dec 2025, 61 month-end
--    points -> 60 months of returns) for Crunch Capital Advisors' six-fund
--    universe, plus the market benchmark used for CAPM in Lecture 3.
--      CRNQ - Crunch US Equity Fund
--      CRNI - Crunch International Equity Fund
--      CRNB - Crunch Core Bond Fund
--      CRNH - Crunch High-Yield Bond Fund
--      CRNG - Crunch Gold Trust
--      CRNR - Crunch Real Estate Fund
--      MKT  - Crunch Market Composite (benchmark; not investable, CAPM only)
CREATE TABLE asset_prices (
    price_id    INTEGER PRIMARY KEY,
    ticker      TEXT    NOT NULL,
    price_date  DATE    NOT NULL,
    adj_close   NUMERIC NOT NULL
);

INSERT INTO asset_prices (price_id, ticker, price_date, adj_close) VALUES
(1, 'CRNQ', '2020-12-31', 50.00),
(2, 'CRNI', '2020-12-31', 40.00),
(3, 'CRNB', '2020-12-31', 100.00),
(4, 'CRNH', '2020-12-31', 60.00),
(5, 'CRNG', '2020-12-31', 180.00),
(6, 'CRNR', '2020-12-31', 70.00),
(7, 'MKT', '2020-12-31', 4000.00),
(8, 'CRNQ', '2021-01-31', 46.67),
(9, 'CRNI', '2021-01-31', 38.38),
(10, 'CRNB', '2021-01-31', 99.15),
(11, 'CRNH', '2021-01-31', 58.55),
(12, 'CRNG', '2021-01-31', 179.02),
(13, 'CRNR', '2021-01-31', 61.58),
(14, 'MKT', '2021-01-31', 3917.03),
(15, 'CRNQ', '2021-02-28', 46.67),
(16, 'CRNI', '2021-02-28', 39.47),
(17, 'CRNB', '2021-02-28', 100.08),
(18, 'CRNH', '2021-02-28', 59.90),
(19, 'CRNG', '2021-02-28', 179.32),
(20, 'CRNR', '2021-02-28', 62.52),
(21, 'MKT', '2021-02-28', 3951.75),
(22, 'CRNQ', '2021-03-31', 48.55),
(23, 'CRNI', '2021-03-31', 42.26),
(24, 'CRNB', '2021-03-31', 102.91),
(25, 'CRNH', '2021-03-31', 64.32),
(26, 'CRNG', '2021-03-31', 174.34),
(27, 'CRNR', '2021-03-31', 67.62),
(28, 'MKT', '2021-03-31', 4081.64),
(29, 'CRNQ', '2021-04-30', 53.02),
(30, 'CRNI', '2021-04-30', 46.22),
(31, 'CRNB', '2021-04-30', 101.77),
(32, 'CRNH', '2021-04-30', 67.64),
(33, 'CRNG', '2021-04-30', 177.43),
(34, 'CRNR', '2021-04-30', 71.40),
(35, 'MKT', '2021-04-30', 4438.02),
(36, 'CRNQ', '2021-05-31', 57.69),
(37, 'CRNI', '2021-05-31', 49.61),
(38, 'CRNB', '2021-05-31', 101.72),
(39, 'CRNH', '2021-05-31', 70.69),
(40, 'CRNG', '2021-05-31', 172.02),
(41, 'CRNR', '2021-05-31', 70.47),
(42, 'MKT', '2021-05-31', 4667.54),
(43, 'CRNQ', '2021-06-30', 53.80),
(44, 'CRNI', '2021-06-30', 46.18),
(45, 'CRNB', '2021-06-30', 101.93),
(46, 'CRNH', '2021-06-30', 69.41),
(47, 'CRNG', '2021-06-30', 170.10),
(48, 'CRNR', '2021-06-30', 65.31),
(49, 'MKT', '2021-06-30', 4473.06),
(50, 'CRNQ', '2021-07-31', 57.62),
(51, 'CRNI', '2021-07-31', 50.79),
(52, 'CRNB', '2021-07-31', 101.22),
(53, 'CRNH', '2021-07-31', 68.46),
(54, 'CRNG', '2021-07-31', 182.62),
(55, 'CRNR', '2021-07-31', 67.98),
(56, 'MKT', '2021-07-31', 4765.38),
(57, 'CRNQ', '2021-08-31', 51.45),
(58, 'CRNI', '2021-08-31', 49.15),
(59, 'CRNB', '2021-08-31', 101.60),
(60, 'CRNH', '2021-08-31', 65.96),
(61, 'CRNG', '2021-08-31', 191.76),
(62, 'CRNR', '2021-08-31', 65.64),
(63, 'MKT', '2021-08-31', 4388.93),
(64, 'CRNQ', '2021-09-30', 53.74),
(65, 'CRNI', '2021-09-30', 53.08),
(66, 'CRNB', '2021-09-30', 100.98),
(67, 'CRNH', '2021-09-30', 66.68),
(68, 'CRNG', '2021-09-30', 190.13),
(69, 'CRNR', '2021-09-30', 65.19),
(70, 'MKT', '2021-09-30', 4565.73),
(71, 'CRNQ', '2021-10-31', 56.28),
(72, 'CRNI', '2021-10-31', 54.35),
(73, 'CRNB', '2021-10-31', 98.39),
(74, 'CRNH', '2021-10-31', 63.79),
(75, 'CRNG', '2021-10-31', 185.61),
(76, 'CRNR', '2021-10-31', 63.91),
(77, 'MKT', '2021-10-31', 4740.81),
(78, 'CRNQ', '2021-11-30', 57.38),
(79, 'CRNI', '2021-11-30', 54.21),
(80, 'CRNB', '2021-11-30', 99.18),
(81, 'CRNH', '2021-11-30', 63.88),
(82, 'CRNG', '2021-11-30', 189.88),
(83, 'CRNR', '2021-11-30', 67.35),
(84, 'MKT', '2021-11-30', 4873.70),
(85, 'CRNQ', '2021-12-31', 53.61),
(86, 'CRNI', '2021-12-31', 48.27),
(87, 'CRNB', '2021-12-31', 99.92),
(88, 'CRNH', '2021-12-31', 61.89),
(89, 'CRNG', '2021-12-31', 194.42),
(90, 'CRNR', '2021-12-31', 65.00),
(91, 'MKT', '2021-12-31', 4578.98),
(92, 'CRNQ', '2022-01-31', 52.27),
(93, 'CRNI', '2022-01-31', 47.64),
(94, 'CRNB', '2022-01-31', 100.97),
(95, 'CRNH', '2022-01-31', 60.74),
(96, 'CRNG', '2022-01-31', 189.71),
(97, 'CRNR', '2022-01-31', 66.71),
(98, 'MKT', '2022-01-31', 4445.91),
(99, 'CRNQ', '2022-02-28', 54.84),
(100, 'CRNI', '2022-02-28', 49.55),
(101, 'CRNB', '2022-02-28', 100.36),
(102, 'CRNH', '2022-02-28', 62.42),
(103, 'CRNG', '2022-02-28', 186.67),
(104, 'CRNR', '2022-02-28', 65.76),
(105, 'MKT', '2022-02-28', 4421.57),
(106, 'CRNQ', '2022-03-31', 55.90),
(107, 'CRNI', '2022-03-31', 51.66),
(108, 'CRNB', '2022-03-31', 100.25),
(109, 'CRNH', '2022-03-31', 62.76),
(110, 'CRNG', '2022-03-31', 187.21),
(111, 'CRNR', '2022-03-31', 70.15),
(112, 'MKT', '2022-03-31', 4674.51),
(113, 'CRNQ', '2022-04-30', 52.33),
(114, 'CRNI', '2022-04-30', 46.09),
(115, 'CRNB', '2022-04-30', 104.03),
(116, 'CRNH', '2022-04-30', 63.83),
(117, 'CRNG', '2022-04-30', 189.25),
(118, 'CRNR', '2022-04-30', 69.63),
(119, 'MKT', '2022-04-30', 4432.93),
(120, 'CRNQ', '2022-05-31', 51.61),
(121, 'CRNI', '2022-05-31', 44.47),
(122, 'CRNB', '2022-05-31', 103.78),
(123, 'CRNH', '2022-05-31', 62.17),
(124, 'CRNG', '2022-05-31', 195.01),
(125, 'CRNR', '2022-05-31', 75.25),
(126, 'MKT', '2022-05-31', 4337.07),
(127, 'CRNQ', '2022-06-30', 52.00),
(128, 'CRNI', '2022-06-30', 42.86),
(129, 'CRNB', '2022-06-30', 102.41),
(130, 'CRNH', '2022-06-30', 60.44),
(131, 'CRNG', '2022-06-30', 187.28),
(132, 'CRNR', '2022-06-30', 75.64),
(133, 'MKT', '2022-06-30', 4418.97),
(134, 'CRNQ', '2022-07-31', 53.19),
(135, 'CRNI', '2022-07-31', 43.73),
(136, 'CRNB', '2022-07-31', 105.18),
(137, 'CRNH', '2022-07-31', 61.29),
(138, 'CRNG', '2022-07-31', 187.73),
(139, 'CRNR', '2022-07-31', 79.77),
(140, 'MKT', '2022-07-31', 4453.78),
(141, 'CRNQ', '2022-08-31', 51.01),
(142, 'CRNI', '2022-08-31', 44.25),
(143, 'CRNB', '2022-08-31', 108.12),
(144, 'CRNH', '2022-08-31', 61.47),
(145, 'CRNG', '2022-08-31', 187.00),
(146, 'CRNR', '2022-08-31', 74.68),
(147, 'MKT', '2022-08-31', 4400.13),
(148, 'CRNQ', '2022-09-30', 52.92),
(149, 'CRNI', '2022-09-30', 44.78),
(150, 'CRNB', '2022-09-30', 109.34),
(151, 'CRNH', '2022-09-30', 63.87),
(152, 'CRNG', '2022-09-30', 196.27),
(153, 'CRNR', '2022-09-30', 81.48),
(154, 'MKT', '2022-09-30', 4385.98),
(155, 'CRNQ', '2022-10-31', 56.53),
(156, 'CRNI', '2022-10-31', 46.20),
(157, 'CRNB', '2022-10-31', 107.51),
(158, 'CRNH', '2022-10-31', 63.40),
(159, 'CRNG', '2022-10-31', 207.12),
(160, 'CRNR', '2022-10-31', 79.72),
(161, 'MKT', '2022-10-31', 4529.22),
(162, 'CRNQ', '2022-11-30', 58.09),
(163, 'CRNI', '2022-11-30', 45.45),
(164, 'CRNB', '2022-11-30', 107.64),
(165, 'CRNH', '2022-11-30', 63.33),
(166, 'CRNG', '2022-11-30', 202.70),
(167, 'CRNR', '2022-11-30', 84.72),
(168, 'MKT', '2022-11-30', 4666.44),
(169, 'CRNQ', '2022-12-31', 62.64),
(170, 'CRNI', '2022-12-31', 49.39),
(171, 'CRNB', '2022-12-31', 109.56),
(172, 'CRNH', '2022-12-31', 67.21),
(173, 'CRNG', '2022-12-31', 198.89),
(174, 'CRNR', '2022-12-31', 85.50),
(175, 'MKT', '2022-12-31', 4981.53),
(176, 'CRNQ', '2023-01-31', 67.72),
(177, 'CRNI', '2023-01-31', 52.64),
(178, 'CRNB', '2023-01-31', 106.53),
(179, 'CRNH', '2023-01-31', 65.51),
(180, 'CRNG', '2023-01-31', 206.47),
(181, 'CRNR', '2023-01-31', 89.02),
(182, 'MKT', '2023-01-31', 5177.83),
(183, 'CRNQ', '2023-02-28', 70.37),
(184, 'CRNI', '2023-02-28', 53.97),
(185, 'CRNB', '2023-02-28', 103.55),
(186, 'CRNH', '2023-02-28', 64.52),
(187, 'CRNG', '2023-02-28', 211.22),
(188, 'CRNR', '2023-02-28', 92.72),
(189, 'MKT', '2023-02-28', 5442.22),
(190, 'CRNQ', '2023-03-31', 76.00),
(191, 'CRNI', '2023-03-31', 57.92),
(192, 'CRNB', '2023-03-31', 106.73),
(193, 'CRNH', '2023-03-31', 70.88),
(194, 'CRNG', '2023-03-31', 220.48),
(195, 'CRNR', '2023-03-31', 96.85),
(196, 'MKT', '2023-03-31', 5852.19),
(197, 'CRNQ', '2023-04-30', 77.18),
(198, 'CRNI', '2023-04-30', 57.12),
(199, 'CRNB', '2023-04-30', 104.20),
(200, 'CRNH', '2023-04-30', 71.70),
(201, 'CRNG', '2023-04-30', 220.23),
(202, 'CRNR', '2023-04-30', 99.78),
(203, 'MKT', '2023-04-30', 5973.01),
(204, 'CRNQ', '2023-05-31', 74.85),
(205, 'CRNI', '2023-05-31', 56.26),
(206, 'CRNB', '2023-05-31', 105.06),
(207, 'CRNH', '2023-05-31', 71.98),
(208, 'CRNG', '2023-05-31', 223.70),
(209, 'CRNR', '2023-05-31', 93.26),
(210, 'MKT', '2023-05-31', 5966.91),
(211, 'CRNQ', '2023-06-30', 71.61),
(212, 'CRNI', '2023-06-30', 55.28),
(213, 'CRNB', '2023-06-30', 106.99),
(214, 'CRNH', '2023-06-30', 73.18),
(215, 'CRNG', '2023-06-30', 219.80),
(216, 'CRNR', '2023-06-30', 86.29),
(217, 'MKT', '2023-06-30', 5670.64),
(218, 'CRNQ', '2023-07-31', 74.84),
(219, 'CRNI', '2023-07-31', 54.76),
(220, 'CRNB', '2023-07-31', 110.02),
(221, 'CRNH', '2023-07-31', 79.44),
(222, 'CRNG', '2023-07-31', 223.07),
(223, 'CRNR', '2023-07-31', 87.06),
(224, 'MKT', '2023-07-31', 5808.57),
(225, 'CRNQ', '2023-08-31', 77.38),
(226, 'CRNI', '2023-08-31', 61.27),
(227, 'CRNB', '2023-08-31', 109.76),
(228, 'CRNH', '2023-08-31', 81.44),
(229, 'CRNG', '2023-08-31', 209.17),
(230, 'CRNR', '2023-08-31', 93.57),
(231, 'MKT', '2023-08-31', 6107.11),
(232, 'CRNQ', '2023-09-30', 74.83),
(233, 'CRNI', '2023-09-30', 57.56),
(234, 'CRNB', '2023-09-30', 111.65),
(235, 'CRNH', '2023-09-30', 81.49),
(236, 'CRNG', '2023-09-30', 208.14),
(237, 'CRNR', '2023-09-30', 89.74),
(238, 'MKT', '2023-09-30', 6054.16),
(239, 'CRNQ', '2023-10-31', 69.45),
(240, 'CRNI', '2023-10-31', 56.96),
(241, 'CRNB', '2023-10-31', 112.83),
(242, 'CRNH', '2023-10-31', 80.82),
(243, 'CRNG', '2023-10-31', 219.71),
(244, 'CRNR', '2023-10-31', 87.34),
(245, 'MKT', '2023-10-31', 5835.03),
(246, 'CRNQ', '2023-11-30', 70.79),
(247, 'CRNI', '2023-11-30', 57.12),
(248, 'CRNB', '2023-11-30', 112.38),
(249, 'CRNH', '2023-11-30', 81.31),
(250, 'CRNG', '2023-11-30', 232.98),
(251, 'CRNR', '2023-11-30', 83.31),
(252, 'MKT', '2023-11-30', 5897.55),
(253, 'CRNQ', '2023-12-31', 69.83),
(254, 'CRNI', '2023-12-31', 60.07),
(255, 'CRNB', '2023-12-31', 112.35),
(256, 'CRNH', '2023-12-31', 79.80),
(257, 'CRNG', '2023-12-31', 238.85),
(258, 'CRNR', '2023-12-31', 83.46),
(259, 'MKT', '2023-12-31', 5969.64),
(260, 'CRNQ', '2024-01-31', 75.83),
(261, 'CRNI', '2024-01-31', 65.52),
(262, 'CRNB', '2024-01-31', 110.28),
(263, 'CRNH', '2024-01-31', 76.18),
(264, 'CRNG', '2024-01-31', 251.71),
(265, 'CRNR', '2024-01-31', 91.00),
(266, 'MKT', '2024-01-31', 6544.17),
(267, 'CRNQ', '2024-02-29', 76.62),
(268, 'CRNI', '2024-02-29', 64.84),
(269, 'CRNB', '2024-02-29', 110.90),
(270, 'CRNH', '2024-02-29', 77.85),
(271, 'CRNG', '2024-02-29', 245.77),
(272, 'CRNR', '2024-02-29', 92.43),
(273, 'MKT', '2024-02-29', 6412.91),
(274, 'CRNQ', '2024-03-31', 76.95),
(275, 'CRNI', '2024-03-31', 65.65),
(276, 'CRNB', '2024-03-31', 109.40),
(277, 'CRNH', '2024-03-31', 78.12),
(278, 'CRNG', '2024-03-31', 252.64),
(279, 'CRNR', '2024-03-31', 93.12),
(280, 'MKT', '2024-03-31', 6324.02),
(281, 'CRNQ', '2024-04-30', 73.12),
(282, 'CRNI', '2024-04-30', 62.17),
(283, 'CRNB', '2024-04-30', 109.35),
(284, 'CRNH', '2024-04-30', 77.31),
(285, 'CRNG', '2024-04-30', 254.65),
(286, 'CRNR', '2024-04-30', 96.40),
(287, 'MKT', '2024-04-30', 5941.07),
(288, 'CRNQ', '2024-05-31', 77.54),
(289, 'CRNI', '2024-05-31', 64.18),
(290, 'CRNB', '2024-05-31', 106.92),
(291, 'CRNH', '2024-05-31', 77.27),
(292, 'CRNG', '2024-05-31', 239.89),
(293, 'CRNR', '2024-05-31', 91.86),
(294, 'MKT', '2024-05-31', 6167.29),
(295, 'CRNQ', '2024-06-30', 79.47),
(296, 'CRNI', '2024-06-30', 65.35),
(297, 'CRNB', '2024-06-30', 108.92),
(298, 'CRNH', '2024-06-30', 79.35),
(299, 'CRNG', '2024-06-30', 247.46),
(300, 'CRNR', '2024-06-30', 92.02),
(301, 'MKT', '2024-06-30', 6273.95),
(302, 'CRNQ', '2024-07-31', 72.19),
(303, 'CRNI', '2024-07-31', 58.11),
(304, 'CRNB', '2024-07-31', 109.33),
(305, 'CRNH', '2024-07-31', 76.89),
(306, 'CRNG', '2024-07-31', 244.58),
(307, 'CRNR', '2024-07-31', 93.71),
(308, 'MKT', '2024-07-31', 5666.66),
(309, 'CRNQ', '2024-08-31', 69.98),
(310, 'CRNI', '2024-08-31', 54.41),
(311, 'CRNB', '2024-08-31', 110.12),
(312, 'CRNH', '2024-08-31', 76.22),
(313, 'CRNG', '2024-08-31', 238.99),
(314, 'CRNR', '2024-08-31', 94.48),
(315, 'MKT', '2024-08-31', 5623.60),
(316, 'CRNQ', '2024-09-30', 74.09),
(317, 'CRNI', '2024-09-30', 56.81),
(318, 'CRNB', '2024-09-30', 108.50),
(319, 'CRNH', '2024-09-30', 76.82),
(320, 'CRNG', '2024-09-30', 234.04),
(321, 'CRNR', '2024-09-30', 96.20),
(322, 'MKT', '2024-09-30', 5802.60),
(323, 'CRNQ', '2024-10-31', 77.27),
(324, 'CRNI', '2024-10-31', 61.79),
(325, 'CRNB', '2024-10-31', 110.34),
(326, 'CRNH', '2024-10-31', 79.28),
(327, 'CRNG', '2024-10-31', 249.04),
(328, 'CRNR', '2024-10-31', 103.53),
(329, 'MKT', '2024-10-31', 5999.15),
(330, 'CRNQ', '2024-11-30', 77.93),
(331, 'CRNI', '2024-11-30', 62.52),
(332, 'CRNB', '2024-11-30', 111.65),
(333, 'CRNH', '2024-11-30', 79.31),
(334, 'CRNG', '2024-11-30', 237.17),
(335, 'CRNR', '2024-11-30', 104.60),
(336, 'MKT', '2024-11-30', 6066.18),
(337, 'CRNQ', '2024-12-31', 77.50),
(338, 'CRNI', '2024-12-31', 65.27),
(339, 'CRNB', '2024-12-31', 114.48),
(340, 'CRNH', '2024-12-31', 80.70),
(341, 'CRNG', '2024-12-31', 256.23),
(342, 'CRNR', '2024-12-31', 95.75),
(343, 'MKT', '2024-12-31', 6161.80),
(344, 'CRNQ', '2025-01-31', 78.11),
(345, 'CRNI', '2025-01-31', 62.58),
(346, 'CRNB', '2025-01-31', 113.46),
(347, 'CRNH', '2025-01-31', 80.25),
(348, 'CRNG', '2025-01-31', 255.22),
(349, 'CRNR', '2025-01-31', 95.86),
(350, 'MKT', '2025-01-31', 6041.59),
(351, 'CRNQ', '2025-02-28', 76.34),
(352, 'CRNI', '2025-02-28', 61.60),
(353, 'CRNB', '2025-02-28', 113.95),
(354, 'CRNH', '2025-02-28', 80.38),
(355, 'CRNG', '2025-02-28', 255.93),
(356, 'CRNR', '2025-02-28', 98.32),
(357, 'MKT', '2025-02-28', 5941.98),
(358, 'CRNQ', '2025-03-31', 73.23),
(359, 'CRNI', '2025-03-31', 59.07),
(360, 'CRNB', '2025-03-31', 117.61),
(361, 'CRNH', '2025-03-31', 81.66),
(362, 'CRNG', '2025-03-31', 249.86),
(363, 'CRNR', '2025-03-31', 93.64),
(364, 'MKT', '2025-03-31', 5718.19),
(365, 'CRNQ', '2025-04-30', 76.27),
(366, 'CRNI', '2025-04-30', 62.07),
(367, 'CRNB', '2025-04-30', 119.29),
(368, 'CRNH', '2025-04-30', 83.68),
(369, 'CRNG', '2025-04-30', 248.84),
(370, 'CRNR', '2025-04-30', 103.16),
(371, 'MKT', '2025-04-30', 6010.92),
(372, 'CRNQ', '2025-05-31', 81.34),
(373, 'CRNI', '2025-05-31', 63.49),
(374, 'CRNB', '2025-05-31', 120.05),
(375, 'CRNH', '2025-05-31', 86.92),
(376, 'CRNG', '2025-05-31', 245.36),
(377, 'CRNR', '2025-05-31', 98.97),
(378, 'MKT', '2025-05-31', 6361.22),
(379, 'CRNQ', '2025-06-30', 79.14),
(380, 'CRNI', '2025-06-30', 68.69),
(381, 'CRNB', '2025-06-30', 117.95),
(382, 'CRNH', '2025-06-30', 84.68),
(383, 'CRNG', '2025-06-30', 257.73),
(384, 'CRNR', '2025-06-30', 104.89),
(385, 'MKT', '2025-06-30', 6200.01),
(386, 'CRNQ', '2025-07-31', 83.35),
(387, 'CRNI', '2025-07-31', 70.39),
(388, 'CRNB', '2025-07-31', 117.38),
(389, 'CRNH', '2025-07-31', 89.59),
(390, 'CRNG', '2025-07-31', 251.79),
(391, 'CRNR', '2025-07-31', 108.61),
(392, 'MKT', '2025-07-31', 6428.24),
(393, 'CRNQ', '2025-08-31', 82.37),
(394, 'CRNI', '2025-08-31', 65.74),
(395, 'CRNB', '2025-08-31', 120.77),
(396, 'CRNH', '2025-08-31', 90.68),
(397, 'CRNG', '2025-08-31', 233.36),
(398, 'CRNR', '2025-08-31', 109.87),
(399, 'MKT', '2025-08-31', 6214.56),
(400, 'CRNQ', '2025-09-30', 86.80),
(401, 'CRNI', '2025-09-30', 72.05),
(402, 'CRNB', '2025-09-30', 123.50),
(403, 'CRNH', '2025-09-30', 97.52),
(404, 'CRNG', '2025-09-30', 250.70),
(405, 'CRNR', '2025-09-30', 105.72),
(406, 'MKT', '2025-09-30', 6549.43),
(407, 'CRNQ', '2025-10-31', 88.20),
(408, 'CRNI', '2025-10-31', 71.50),
(409, 'CRNB', '2025-10-31', 124.47),
(410, 'CRNH', '2025-10-31', 97.56),
(411, 'CRNG', '2025-10-31', 263.30),
(412, 'CRNR', '2025-10-31', 102.06),
(413, 'MKT', '2025-10-31', 6462.81),
(414, 'CRNQ', '2025-11-30', 86.79),
(415, 'CRNI', '2025-11-30', 68.42),
(416, 'CRNB', '2025-11-30', 126.09),
(417, 'CRNH', '2025-11-30', 97.32),
(418, 'CRNG', '2025-11-30', 265.30),
(419, 'CRNR', '2025-11-30', 100.54),
(420, 'MKT', '2025-11-30', 6581.89),
(421, 'CRNQ', '2025-12-31', 92.47),
(422, 'CRNI', '2025-12-31', 72.57),
(423, 'CRNB', '2025-12-31', 125.34),
(424, 'CRNH', '2025-12-31', 95.94),
(425, 'CRNG', '2025-12-31', 259.14),
(426, 'CRNR', '2025-12-31', 100.78),
(427, 'MKT', '2025-12-31', 6952.95);

-- 2. A one-row reference table: the risk-free rate and the assumed equity
--    risk premium used for CAPM in Lecture 3, as of the valuation date.
CREATE TABLE capital_market_assumptions (
    assumption_set      TEXT    PRIMARY KEY,
    valuation_date       DATE    NOT NULL,
    risk_free_rate         NUMERIC NOT NULL,   -- 10-year Treasury proxy, as a decimal
    equity_risk_premium    NUMERIC NOT NULL    -- assumed/supplied ERP, as a decimal
);

INSERT INTO capital_market_assumptions VALUES
('base_case', '2025-12-31', 0.0300, 0.0550);
```

Sanity checks — these should print `427` and `1`:

```sql
SELECT COUNT(*) FROM asset_prices;
SELECT COUNT(*) FROM capital_market_assumptions;
```

```sql
-- One row per ticker, each with 61 monthly prices spanning the full window
SELECT ticker, COUNT(*), MIN(price_date), MAX(price_date)
FROM asset_prices
GROUP BY ticker
ORDER BY ticker;
```

`asset_prices` is flat and denormalized on purpose — same philosophy as `stock_prices` in Week 4, just with seven tickers instead of two. `capital_market_assumptions` holds the risk-free rate (3.00%) and equity risk premium (5.50%) you'll plug into CAPM in Lecture 3 — note that these are **supplied assumptions**, not something derived from the six funds' own realized returns, and Lecture 3 makes a point of showing you both numbers side by side so you can see how far apart "what CAPM says you should have expected" and "what actually happened over this five-year window" can be.

## Weekly schedule

The schedule below adds up to approximately **27 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed the data; risk, return, and covariance | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Mean-variance optimization; tracing the frontier | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | GMV and max-Sharpe portfolios; CAPM and betas | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Thursday | Constrained allocation challenge | 0h | 0h | 1.5h | 0.5h | 1.5h | 1h | 4.5h |
| Friday | Diversification stress-test challenge | 0h | 0h | 1.5h | 0.5h | 1h | 1.5h | 4.5h |
| Saturday | Mini-project (optimize a multi-asset allocation) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **4h** | **3h** | **3.5h** | **5.5h** | **5h** | **27h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-risk-return-and-covariance.md](./lecture-notes/01-risk-return-and-covariance.md) | Expected return, variance, covariance, correlation, and why diversification actually reduces risk | 2h |
| 2 | [lecture-notes/02-the-efficient-frontier.md](./lecture-notes/02-the-efficient-frontier.md) | Mean-variance optimization, the efficient frontier, GMV and max-Sharpe portfolios in closed form | 2h |
| 3 | [lecture-notes/03-capm-and-factor-tilts.md](./lecture-notes/03-capm-and-factor-tilts.md) | The capital market line, betas, CAPM-required returns, and adding factor tilts to an allocation | 2h |
| 4 | [exercises/exercise-01-covariance-from-returns.md](./exercises/exercise-01-covariance-from-returns.md) | Build the full 6×6 covariance and correlation matrix from `asset_prices` | 1.5h |
| 5 | [exercises/exercise-02-trace-the-frontier.md](./exercises/exercise-02-trace-the-frontier.md) | Solve for the efficient frontier and plot expected return vs. volatility | 1.5h |
| 6 | [exercises/exercise-03-max-sharpe-portfolio.md](./exercises/exercise-03-max-sharpe-portfolio.md) | Solve for the GMV and tangency (max-Sharpe) portfolios in closed form | 1.5h |
| 7 | [challenges/challenge-01-constrained-allocation.md](./challenges/challenge-01-constrained-allocation.md) | Re-optimize under no-short and position-cap constraints; compare to the unconstrained solution | 1.5h |
| 8 | [challenges/challenge-02-diversification-stress-test.md](./challenges/challenge-02-diversification-stress-test.md) | Stress-test the "optimal" portfolio's diversification benefit in a correlation-spike crash scenario | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Optimize a multi-asset allocation on the efficient frontier and report the max-Sharpe weights with their risk profile | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, free data sources, and further reading | — |

## By the end of this week you can…

- Turn a table of raw prices into returns, variances, and a full covariance matrix — in SQL and pandas, never a spreadsheet.
- Compute the expected return and volatility of *any* set of portfolio weights against that covariance matrix, and explain in one sentence why a portfolio's variance is not the weighted average of its assets' variances.
- Trace the efficient frontier and pinpoint, in closed form, the global-minimum-variance and maximum-Sharpe portfolios for a real (if fictional) six-fund universe.
- Price each asset's CAPM-required return from a regression beta, and reason about tilting an allocation toward or away from a given exposure.
- Push back, with numbers, on a naive "just run the optimizer" allocation — knowing where mean-variance optimization is fragile, and what a constrained or stress-tested version looks like instead.

## Up next

Week 9 — Investment analytics & rules-based backtesting — now that you can build and defend a static allocation, we turn to testing whether a *rule* for changing that allocation over time would actually have worked, net of costs.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
