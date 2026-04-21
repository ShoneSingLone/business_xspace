# Boundless Static Business xspace

This is an open-source project based on xspace, designed to provide a powerful and flexible API management platform. The
project integrates modern web technologies and offers a wide range of features, including API documentation management,
test tools, Mock services, and more.

## Key Features

- **API Management**: Supports creating, editing, deleting, and categorizing APIs.
- **Documentation Generation**: Automatically generates API documentation in Markdown and HTML formats.
- **Testing Tools**: Provides API testing capabilities supporting multiple request methods and parameter types.
- **Mock Services**: Supports generating Mock data, facilitating front-end development and testing.
- **Multi-language Support**: Offers interface language switching for multiple languages.
- **Permission Management**: Supports user permission management to ensure data security.
- **Integration Tools**: Integrated with CI/CD tools for automated deployment and build processes.

## Installation Guide

### System Requirements

- Node.js (v14.x or higher)
- npm or yarn
- MongoDB (for data storage)

### Installation Steps

1. **Clone the Repository**:

    ```bash
    git clone https://gitee.com/ShoneSingLone/boundless_static_business_xspace.git
    cd boundless_static_business_xspace
    ```

2. **Install Dependencies**:

    ```bash
    npm install
    # or using yarn
    yarn install
    ```

3. **Configure the Database**: Ensure MongoDB is installed and running, then create a `.env` file in the project root
   directory and configure the database connection:

    ```env
    MONGO_URI=mongodb://localhost:27017/xspace
    ```

4. **Start the Project**:

    ```bash
    npm run dev
    # or using yarn
    yarn dev
    ```

5. **Access the Project**: Open a browser and navigate to `http://localhost:3000` to view the xspace interface.

## Usage Instructions

### API Management

- **Creating an API**: On the project page, click the "New API" button and fill in the basic information, request
  parameters, response examples, etc.
- **Editing an API**: Click the API name to enter the API detail page, where you can edit different sections of the API.
- **Deleting an API**: On the API detail page, click the "Delete" button to remove the API.

### Documentation Generation

- **Export Documentation**: On the API detail page, click the "Export" button and choose to export as Markdown or HTML
  format.
- **Online Viewing**: On the project page, click the "Documentation" button to view the generated API documentation
  online.

### Testing Features

- **API Testing**: On the API detail page, click the "Test" button to send requests and view response results.
- **Mock Services**: On the API detail page, click the "Mock" button to generate Mock data and perform tests.

### Multi-language Support

- **Switching Languages**: On the top-right corner of the project page, click the language switch button to select a
  different interface language.

### Permission Management

- **User Management**: In the project settings, you can manage user permissions and assign different roles and access
  rights.

### Integration Tools

- **CI/CD Integration**: In the project settings, you can configure CI/CD tools for automated deployment and building.

## Contribution Guide

We welcome contributions of code and documentation! Please follow these steps:

1. **Fork the Repository**: Fork this repository on Gitee.
2. **Create a Branch**: Create a new branch for development.
3. **Commit Changes**: Commit your code with clear and descriptive commit messages.
4. **Submit a Pull Request**: Submit a Pull Request on Gitee and wait for review and merging.

## License

This project uses the MIT License. For details, please refer to the [LICENSE](LICENSE) file.

## Contact Us

If you have any questions or suggestions, please submit an Issue on Gitee or contact the project maintainers.
