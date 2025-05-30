# Руководство по использованию системы книг

## Описание
Система книг — это DataviewJS-скрипт для Obsidian, который отображает список книг в виде таблицы с фильтрацией, сортировкой и стилизованным интерфейсом.

## Установка
1. Сохраните скрипт в файле с расширением `.js` в вашей папке Obsidian.
2. Убедитесь, что путь к книгам в конфигурации (`CONFIG.PATHS.BOOKS`) соответствует вашей структуре: `"3. Resourses/3. Книги"`.
3. В заметке Obsidian добавьте код:
   ```dataviewjs
   dv.executeJs("путь/к/вашему/скрипту.js");
   ```

## Использование
- **Фильтрация**:
  - Используйте выпадающий список для выбора категории (тега).
  - Введите запрос в поле поиска для фильтрации по названию книги.
- **Сортировка**:
  - Кликните на заголовок столбца (Книга, Автор, Рейтинг, Статус, Дата) для сортировки.
  - Повторный клик меняет направление сортировки (по возрастанию/убыванию).
- **Просмотр**:
  - Таблица отображает обложку, название, автора, категории, рейтинг, статус и дату изменения.
  - Клик по названию книги открывает связанную заметку.

## Конфигурация
Настройки находятся в объекте `CONFIG` в начале скрипта:
- `PATHS.BOOKS`: путь к папке с книгами.
- `TAGS.BASE`: базовый тег (`book`).
- `UI.STYLES`: стили для элементов интерфейса.
- `UI.TEXT`: текстовые подписи (плейсхолдеры, сообщение об отсутствии книг).
- `TABLE.HEADERS`: заголовки таблицы.
- `TABLE.STATUS_COLORS`: цвета для статусов.
- `SORT`: настройки сортировки по умолчанию.

## Примечания
- Если книги не найдены, отображается сообщение «Книги не найдены».
- Для корректной работы каждая книга должна иметь тег `book` и обложку (`cover`).

## Пример структуры заметки книги
```markdown
---
tags: [book, fantasy]
cover: путь/к/обложке.jpg
author: Имя Автора
rating: 4
status: В процессе
modified: 2025-05-30
---
```

```js
const CONFIG = {
    PATHS: {
        BOOKS: '"3. Resourses/3. Книги"',
    },
    TAGS: {
        BASE: 'book',
    },
    UI: {
        STYLES: {
            wrapper: `
                display: flex;
                gap: 1rem;
                margin-bottom: 1.5rem;
                padding: 1rem;
                background: var(--background-secondary);
                border-radius: 8px;
                border: 1px solid var(--background-modifier-border);
            `,
            select: `
                padding: 0.5rem;
                border-radius: 6px;
                border: 1px solid var(--background-modifier-border);
                background: var(--background-primary);
                color: var(--text-normal);
                min-width: 150px;
            `,
            input: `
                flex: 1;
                padding: 0.5rem;
                border-radius: 6px;
                border: 1px solid var(--background-modifier-border);
                background: var(--background-primary);
                color: var(--text-normal);
            `,
            table: `
                width: 100%;
                border-collapse: collapse;
                background: var(--background-primary);
                border-radius: 8px;
                overflow: hidden;
                box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            `,
            headerRow: `
                background: var(--background-secondary);
                border-bottom: 2px solid var(--background-modifier-border);
            `,
            headerCell: `
                padding: 1rem;
                text-align: left;
                font-weight: 600;
                color: var(--text-normal);
            `,
            row: `
                border-bottom: 1px solid var(--background-modifier-border);
                transition: background-color 0.2s ease;
            `,
            cell: `
                padding: 1rem;
                vertical-align: top;
            `,
            noResults: `
                padding: 2rem;
                text-align: center;
                color: var(--text-muted);
                font-style: italic;
            `,
        },
        TEXT: {
            selectPlaceholder: 'Все категории',
            searchPlaceholder: '🔍 Поиск книги...',
            noResults: 'Книги не найдены',
        },
    },
    TABLE: {
        HEADERS: [
            { text: '📕 Обложка', field: null },
            { text: '📘 Книга', field: 'title' },
            { text: '✍️ Автор', field: 'author' },
            { text: '📚 Категория', field: null },
            { text: '⭐ Рейтинг', field: 'rating' },
            { text: '📊 Статус', field: 'status' },
            { text: '📅 Дата', field: 'modified' },
        ],
        STATUS_COLORS: {
            'В процессе': '#3b82f6',
            'Завершенные': '#10b981',
            'В планах': '#f59e0b',
        },
    },
    SORT: {
        DEFAULT_FIELD: null,
        DEFAULT_DIRECTION: 'asc',
    },
};

class BookDataService {
    constructor() {
        this.pages = dv.pages(CONFIG.PATHS.BOOKS).where(p => p.cover);
    }

    getFilteredTags() {
        const tags = new Set();
        this.pages.forEach(page => {
            this.normalizeTags(page.tags).forEach(tag => {
                if (tag && tag !== CONFIG.TAGS.BASE) tags.add(tag);
            });
        });
        return Array.from(tags).sort();
    }

    normalizeTags(tags) {
        if (!tags) return [];
        return Array.isArray(tags) ? tags : [tags];
    }

    filterPages(filter) {
        return this.pages.filter(page => this.isPageMatchingFilters(page, filter));
    }

    isPageMatchingFilters(page, { tag, search }) {
        const tagMatch = !tag || this.normalizeTags(page.tags)
            .filter(t => t !== CONFIG.TAGS.BASE)
            .includes(tag);

        const searchMatch = !search ||
            page.file.name.toLowerCase().includes(search.toLowerCase());

        return tagMatch && searchMatch;
    }

    sortPages(pages, sortConfig) {
        if (!sortConfig.field) return pages;

        return [...pages].sort((a, b) => {
            let valueA, valueB;

            switch (sortConfig.field) {
                case 'title':
                    valueA = a.file.name.toLowerCase();
                    valueB = b.file.name.toLowerCase();
                    break;
                case 'author':
                    valueA = (a.author || '').toLowerCase();
                    valueB = (b.author || '').toLowerCase();
                    break;
                case 'rating':
                    valueA = a.rating || 0;
                    valueB = b.rating || 0;
                    break;
                case 'status':
                    valueA = (a.status || '').toLowerCase();
                    valueB = (b.status || '').toLowerCase();
                    break;
                case 'modified':
                    valueA = a.modified || a.created || new Date(0);
                    valueB = b.modified || b.created || new Date(0);
                    break;
                default:
                    return 0;
            }

            return valueA < valueB
                ? sortConfig.direction === 'asc' ? -1 : 1
                : valueA > valueB
                    ? sortConfig.direction === 'asc' ? 1 : -1
                    : 0;
        });
    }
}

class BookUIRenderer {
    constructor(container, onFilterChange, onSortChange) {
        this.container = container;
        this.onFilterChange = onFilterChange;
        this.onSortChange = onSortChange;
        this.dataService = new BookDataService();
    }

    createSelectElement() {
        const select = document.createElement('select');
        select.style.cssText = CONFIG.UI.STYLES.select;

        const defaultOption = document.createElement('option');
        defaultOption.textContent = CONFIG.UI.TEXT.selectPlaceholder;
        defaultOption.value = '';
        select.appendChild(defaultOption);

        this.dataService.getFilteredTags().forEach(tag => {
            const option = document.createElement('option');
            option.textContent = tag;
            option.value = tag;
            select.appendChild(option);
        });

        select.onchange = () => this.onFilterChange({ tag: select.value });

        return select;
    }

    createSearchInput() {
        const input = document.createElement('input');
        input.type = 'text';
        input.placeholder = CONFIG.UI.TEXT.searchPlaceholder;
        input.style.cssText = CONFIG.UI.STYLES.input;

        input.oninput = () => this.onFilterChange({ search: input.value });

        return input;
    }

    createTableHeader(sortConfig) {
        const headerRow = document.createElement('tr');
        headerRow.style.cssText = CONFIG.UI.STYLES.headerRow;

        CONFIG.TABLE.HEADERS.forEach(header => {
            const th = document.createElement('th');
            th.textContent = header.text;
            th.style.cssText = CONFIG.UI.STYLES.headerCell +
                (header.field ? 'cursor: pointer; user-select: none;' : '');

            if (header.field && sortConfig.field === header.field) {
                th.textContent += sortConfig.direction === 'asc' ? ' ↑' : ' ↓';
            }

            if (header.field) {
                th.addEventListener('click', () => this.onSortChange(header.field));
            }

            headerRow.appendChild(th);
        });

        return headerRow;
    }

    createBookRow(page) {
        const row = document.createElement('tr');
        row.style.cssText = CONFIG.UI.STYLES.row;

        row.addEventListener('mouseenter', () => {
            row.style.backgroundColor = 'var(--background-secondary)';
        });
        row.addEventListener('mouseleave', () => {
            row.style.backgroundColor = 'transparent';
        });

        const coverCell = document.createElement('td');
        coverCell.style.cssText = CONFIG.UI.STYLES.cell;
        coverCell.innerHTML = page.cover
            ? `<img src="${page.cover}" width="80" style="border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1);">`
            : '<div style="width: 80px; height: 120px; background: var(--background-secondary); border-radius: 8px; display: flex; align-items: center; justify-content: center; color: var(--text-muted);">—</div>';
        row.appendChild(coverCell);

        const titleCell = document.createElement('td');
        titleCell.style.cssText = CONFIG.UI.STYLES.cell;
        const link = document.createElement('a');
        link.href = page.file.path;
        link.textContent = page.file.name;
        link.className = 'internal-link';
        link.style.cssText = `
            font-weight: 500;
            font-size: 1.1em;
            color: var(--text-accent);
            text-decoration: none;
        `;
        titleCell.appendChild(link);
        row.appendChild(titleCell);

        const authorCell = document.createElement('td');
        authorCell.style.cssText = CONFIG.UI.STYLES.cell + 'color: var(--text-muted);';
        authorCell.textContent = page.author || '–';
        row.appendChild(authorCell);

        const tagsCell = document.createElement('td');
        tagsCell.style.cssText = CONFIG.UI.STYLES.cell;
        const filteredTags = this.dataService.normalizeTags(page.tags)
            .filter(tag => tag !== CONFIG.TAGS.BASE);

        if (filteredTags.length > 0) {
            const tagsContainer = document.createElement('div');
            tagsContainer.style.cssText = 'display: flex; flex-wrap: wrap; gap: 0.5rem;';

            filteredTags.forEach(tag => {
                const tagSpan = document.createElement('span');
                tagSpan.textContent = tag;
                tagSpan.style.cssText = `
                    background: var(--text-accent);
                    color: var(--background-primary);
                    padding: 0.25rem 0.5rem;
                    border-radius: 12px;
                    font-size: 0.85em;
                    font-weight: 500;
                `;
                tagsContainer.appendChild(tagSpan);
            });

            tagsCell.appendChild(tagsContainer);
        } else {
            tagsCell.textContent = '–';
            tagsCell.style.color = 'var(--text-muted)';
        }
        row.appendChild(tagsCell);

        const ratingCell = document.createElement('td');
        ratingCell.style.cssText = CONFIG.UI.STYLES.cell;
        const rating = page.rating || 0;
        const starsContainer = document.createElement('div');
        starsContainer.style.cssText = 'font-size: 1.2em;';

        for (let i = 1; i <= 5; i++) {
            const star = document.createElement('span');
            star.textContent = i <= rating ? '★' : '☆';
            star.style.color = i <= rating ? '#ffd700' : 'var(--text-muted)';
            starsContainer.appendChild(star);
        }
        ratingCell.appendChild(starsContainer);
        row.appendChild(ratingCell);

        const statusCell = document.createElement('td');
        statusCell.style.cssText = CONFIG.UI.STYLES.cell;
        const status = page.status || '';
        if (status) {
            const statusSpan = document.createElement('span');
            statusSpan.textContent = status;
            statusSpan.style.cssText = `
                background: ${CONFIG.TABLE.STATUS_COLORS[status] || 'var(--text-muted)'};
                color: white;
                padding: 0.25rem 0.75rem;
                border-radius: 16px;
                font-size: 0.85em;
                font-weight: 500;
            `;
            statusCell.appendChild(statusSpan);
        } else {
            statusCell.textContent = '–';
            statusCell.style.color = 'var(--text-muted)';
        }
        row.appendChild(statusCell);

        const dateCell = document.createElement('td');
        dateCell.style.cssText = CONFIG.UI.STYLES.cell + 'color: var(--text-muted); font-size: 0.9em;';
        const date = page.modified || page.created;
        dateCell.textContent = date ? date.toFormat('dd.MM.yyyy') : '–';
        row.appendChild(dateCell);

        return row;
    }

    renderNoResults() {
        const div = document.createElement('div');
        div.textContent = CONFIG.UI.TEXT.noResults;
        div.style.cssText = CONFIG.UI.STYLES.noResults;
        return div;
    }

    renderUI() {
        const wrapper = document.createElement('div');
        wrapper.style.cssText = CONFIG.UI.STYLES.wrapper;

        wrapper.appendChild(this.createSelectElement());
        wrapper.appendChild(this.createSearchInput());
        this.container.appendChild(wrapper);
    }

    renderTable(filter, sortConfig) {
        const existingTable = this.container.querySelector('table');
        if (existingTable) existingTable.remove();

        const table = document.createElement('table');
        table.className = 'dataview table-view-table';
        table.style.cssText = CONFIG.UI.STYLES.table;

        table.appendChild(this.createTableHeader(sortConfig));

        const filteredPages = this.dataService.filterPages(filter);
        const sortedPages = this.dataService.sortPages(filteredPages, sortConfig);

        if (sortedPages.length === 0) {
            const noResultsRow = document.createElement('tr');
            const noResultsCell = document.createElement('td');
            noResultsCell.colSpan = CONFIG.TABLE.HEADERS.length;
            noResultsCell.appendChild(this.renderNoResults());
            noResultsRow.appendChild(noResultsCell);
            table.appendChild(noResultsRow);
        } else {
            sortedPages.forEach(page => {
                table.appendChild(this.createBookRow(page));
            });
        }

        this.container.appendChild(table);
    }
}

class BookSystem {
    constructor(container) {
        if (!container) throw new Error('Container is required');
        this.container = container;
        this.filter = { tag: '', search: '' };
        this.sortConfig = { field: CONFIG.SORT.DEFAULT_FIELD, direction: CONFIG.SORT.DEFAULT_DIRECTION };
        this.renderer = new BookUIRenderer(
            container,
            (filterUpdate) => this.updateFilter(filterUpdate),
            (field) => this.handleSort(field)
        );
        this.init();
    }

    init() {
        this.renderer.renderUI();
        this.renderer.renderTable(this.filter, this.sortConfig);
    }

    updateFilter(filterUpdate) {
        this.filter = { ...this.filter, ...filterUpdate };
        this.renderer.renderTable(this.filter, this.sortConfig);
    }

    handleSort(field) {
        if (this.sortConfig.field === field) {
            this.sortConfig.direction = this.sortConfig.direction === 'asc' ? 'desc' : 'asc';
        } else {
            this.sortConfig.field = field;
            this.sortConfig.direction = CONFIG.SORT.DEFAULT_DIRECTION;
        }
        this.renderer.renderTable(this.filter, this.sortConfig);
    }
}

new BookSystem(this.container);
```